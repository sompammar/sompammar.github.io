---
layout: post
title:  "Spring boot proxy application"
date:   2021-09-26 12:33:34 -0700
categories: springboot proxy
---
If you have a need to write a proxy service that can redirect requests to another application, one option is to use nginx service. Other option is to write a controller that can redirect requests to your service.

```
@Controller
public class PodProxyController {
    @Autowired
    IPodInfoService podInfoService;

    final String pathPattern = "/pod/{region}/{pod}/**";

    @RequestMapping(pathPattern)
    public ResponseEntity<?> proxyPath(@PathVariable("region") String region,
                                    @PathVariable("pod") String pod,
                                    @RequestBody(required = false) byte[] body,
                                    HttpMethod method, HttpServletRequest request,
                                    HttpServletResponse response,
                                       @RequestHeader HttpHeaders headers) throws Exception {
        String urlTail = new AntPathMatcher()
                .extractPathWithinPattern(pathPattern, request.getRequestURI() );

        String privateLbrUrl = podInfoService.getPodPrivateLbrUrl(region, pod);
        URI uri = UriComponentsBuilder.fromUriString(privateLbrUrl)
                .path(urlTail)
                .query(request.getQueryString())
                .build(true).toUri();

        Enumeration<String> headerNames = request.getHeaderNames();
        while (headerNames.hasMoreElements()) {
            String headerName = headerNames.nextElement();
            headers.set(headerName, request.getHeader(headerName));
        }

        HttpEntity<?> httpEntity = new HttpEntity<>(body, headers);
        RestTemplate restTemplate = new RestTemplate();
        try {
            return restTemplate.exchange(uri, method, httpEntity, byte[].class);
        } catch(HttpStatusCodeException e) {
            return ResponseEntity.status(e.getRawStatusCode())
                    .headers(e.getResponseHeaders())
                    .body(e.getResponseBodyAsString());
        }
    }
}
```
