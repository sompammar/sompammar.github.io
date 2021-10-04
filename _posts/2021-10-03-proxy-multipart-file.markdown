---
layout: post
title:  "Spring boot proxy application"
date:   2021-09-26 12:33:34 -0700
categories: springboot proxy
---
To implement a proxy to support multipart file uploads following sample can be used

```

@Controller
public class PodProxyController {
    @Autowired
    IPodInfoService podInfoService;

    final String pathPattern = "/pod/{region}/{pod}/**";

    private static final Logger logger = LoggerFactory.getLogger(PodProxyController.class);

    @RequestMapping(pathPattern)
    public ResponseEntity<?> proxyPath(@PathVariable("region") String region,
                                    @PathVariable("pod") String pod,
                                    HttpMethod method, HttpServletRequest request,
                                    HttpServletResponse response, @RequestHeader HttpHeaders headers) throws Exception {

        String requestUri = request.getRequestURI();
        String queryString = request.getQueryString();
        logger.debug("Received proxy req for {}?{}", requestUri, queryString);
        String urlTail = new AntPathMatcher()
                .extractPathWithinPattern(pathPattern, requestUri);
        RestTemplate restTemplate = new RestTemplate();

        String privateLbrUrl = podInfoService.getPodPrivateLbrUrl(region, pod);
        URI uri = UriComponentsBuilder.fromUriString(privateLbrUrl)
                .path(urlTail)
                .query(queryString)
                .build(true).toUri();
        HttpEntity<?> httpEntity = null;

        if (headers.getContentType() != null && headers.getContentType().includes(MediaType.MULTIPART_FORM_DATA)) {
            Collection<Part> parts = request.getParts();
            MultiValueMap<String, Object> body = new LinkedMultiValueMap<>();
            for (Part part: parts) {
                body.add(part.getName(),
                        new MultiPartStreamResource(part.getInputStream(), part.getSubmittedFileName()));
            }
            httpEntity = new HttpEntity<>(body, headers);
        } else {
            logger.debug("Sending req to {} Req Headers {}", uri, headers);
            httpEntity = new HttpEntity<>(new InputStreamResource(request.getInputStream()), headers);
        }
        try {
            return restTemplate.exchange(uri, method, httpEntity, byte[].class);
        } catch (HttpStatusCodeException e) {
            logger.error("Error sending request {} {} Original URL {}", uri,
            headers, requestUri);
            logger.error("Error", e);
            return ResponseEntity.status(e.getRawStatusCode())
                    .headers(e.getResponseHeaders())
                    .body(e.getResponseBodyAsString());
        }
    }

}

```

```
public class MultiPartStreamResource extends InputStreamResource {
    final InputStream inputStream;
    final String filename;

    public MultiPartStreamResource(InputStream inputStream, String filename) throws IOException {
        super(inputStream);
        this.inputStream = inputStream;
        this.filename = filename;
    }

    @Override
    public InputStream getInputStream() throws IOException, IllegalStateException {
        return this.inputStream;
    }

    @Override
    public long contentLength() throws IOException {
        return -1;
    }

    public static byte[] readBytes(InputStream inputStream) throws IOException {
        ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
        IOUtils.copy(inputStream, outputStream);
        return outputStream.toByteArray();
    }

    @Override
    public String getFilename() {
        return filename;
    }
}

```
