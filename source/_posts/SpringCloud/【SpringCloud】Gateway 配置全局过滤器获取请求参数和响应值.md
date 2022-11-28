---
title: 【SpringCloud】Gateway 配置全局过滤器获取请求参数和响应值
tags:
  - SpringCloud
  - Gateway
  - 全局过滤器
categories:
  - SpringCloud
---
# 【SpringCloud】Gateway 配置全局过滤器获取请求参数和响应值

> 实现Ordered接口getOrder()方法，数值越小越靠前执行，记得这一点就OK了。

## 获取请求参数RequestBody

```java
@Component
@Slf4j
@AllArgsConstructor
public class HttpRequestFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();

        String method = request.getMethodValue();
        String contentType = request.getHeaders().getFirst("Content-Type");
        if ("POST".equals(method)) {
            return DataBufferUtils.join(exchange.getRequest().getBody())
                    .flatMap(dataBuffer -> {
                        byte[] bytes = new byte[dataBuffer.readableByteCount()];
                        dataBuffer.read(bytes);
                        try {
                            String bodyString = new String(bytes, "utf-8");
                            log.info(bodyString);//打印请求参数
                            exchange.getAttributes().put("POST_BODY", bodyString);
                        } catch (UnsupportedEncodingException e) {
                            e.printStackTrace();
                        }
                        DataBufferUtils.release(dataBuffer);
                        Flux<DataBuffer> cachedFlux = Flux.defer(() -> {
                            DataBuffer buffer = exchange.getResponse().bufferFactory()
                                    .wrap(bytes);
                            return Mono.just(buffer);
                        });

                        ServerHttpRequest mutatedRequest = new ServerHttpRequestDecorator(
                                exchange.getRequest()) {
                            @Override
                            public Flux<DataBuffer> getBody() {
                                return cachedFlux;
                            }
                        };
                        return chain.filter(exchange.mutate().request(mutatedRequest)
                                .build());
                    });
        }
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return -200;
    }
}
```

## 获取请求响应值ResponseBody

**POSTMAN工具请求里的gzip压缩头导致获取响应值一直乱码，解决gzip压缩后响应值获取**

```java
@Slf4j
@Component
public class HttpResponseFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        String path = request.getPath().toString();
        ServerHttpResponse originalResponse = exchange.getResponse();
        System.out.println(originalResponse.isCommitted());
        DataBufferFactory bufferFactory = originalResponse.bufferFactory();

        ServerHttpResponseDecorator decoratedResponse = new ServerHttpResponseDecorator(originalResponse) {
            @Override
            public Mono<Void> writeWith(Publisher<? extends DataBuffer> body) {

                if (body instanceof Flux) {
                    Flux<? extends DataBuffer> fluxBody = (Flux<? extends DataBuffer>) body;
                    return super.writeWith(fluxBody.buffer().map(dataBuffer -> {
                        DataBufferFactory dataBufferFactory = new DefaultDataBufferFactory();
                        DataBuffer join = dataBufferFactory.join(dataBuffer);
                        byte[] content = new byte[join.readableByteCount()];
                        join.read(content);
                        //释放掉内存
                        DataBufferUtils.release(join);
                        String s = new String(content, StandardCharsets.UTF_8);

                        List<String> strings = exchange.getResponse().getHeaders().get(HttpHeaders.CONTENT_ENCODING);
                        if (!CollectionUtils.isEmpty(strings) && strings.contains("gzip")) {
                            GZIPInputStream gzipInputStream = null;
                            try {
                                gzipInputStream = new GZIPInputStream(new ByteArrayInputStream(content), content.length);
                                StringWriter writer = new StringWriter();
                                IOUtils.copy(gzipInputStream, writer, "UTF-8");
                                s = writer.toString();

                            } catch (IOException e) {
                                log.error("====Gzip IO error", e);
                            } finally {
                                if (gzipInputStream != null) {
                                    try {
                                        gzipInputStream.close();
                                    } catch (IOException e) {
                                        log.error("===Gzip IO close error", e);
                                    }
                                }
                            }
                        } else {
                            s = new String(content, StandardCharsets.UTF_8);
                        }
                        log.info("bodyString: {}", s);//打印请求响应值
                        return bufferFactory.wrap(content);
                    }));
                }
                return super.writeWith(body);
            }
        };
        return chain.filter(exchange.mutate().response(decoratedResponse).build());
    }

    @Override
    public int getOrder() {
        return -200;
    }
}
```
