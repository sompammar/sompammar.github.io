---
layout: post
title:  "register costom message converter springboot"
date:   2018-09-22 21:11:34 -0700
categories: oracle jet oracle data visualization dv
---
To register custom message converter when RestTemplate is used following code can be used

```
private void registerMessageConverter(RestTemplate restTemplate) {
    List<HttpMessageConverter<?>> messageConverters = restTemplate.getMessageConverters();
    for (int i = 0; i < messageConverters.size(); i++) {
        HttpMessageConverter<?> messageConverter = messageConverters.get(i);
        if ( messageConverter.getClass().equals(ResourceHttpMessageConverter.class) )
            messageConverters.set(i, new ResourceHttpMessageConverterHandlingInputStreams());
    }
}


public class ResourceHttpMessageConverterHandlingInputStreams extends ResourceHttpMessageConverter {

    @Override
    protected Long getContentLength(Resource resource, MediaType contentType) throws IOException {
        Long contentLength = super.getContentLength(resource, contentType);

        return contentLength == null || contentLength < 0 ? null : contentLength;
    }
}
```
