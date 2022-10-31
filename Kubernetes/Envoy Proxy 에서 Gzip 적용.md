1. Envoy Gzip Extension
- extensions.filters.http.gzip.v3.Gzip
  - ```
     {
       "memory_level": {...},
       "compression_level": ...,
       "compression_strategy": ...,
       "window_bits": {...},
       "compressor": {...},
       "chunk_size": {...}
     }
     ```
2. Istio Envoy 설정 예시
```
apiVersion: networking.istio.io/v1alpha3
kind: EnvoyFilter
metadata:
  name: wasm-service
  namespace: myns
spec:
  configPatches:
  - applyTo: BOOTSTRAP
    patch:
      operation: MERGE
      value:
        bootstrap_extensions:
        - name: envoy.bootstrap.wasm
          typed_config:
            "@type": type.googleapis.com/envoy.extensions.wasm.v3.WasmService
            singleton: true
            config:
              name: my_plugin
              configuration:
                "@type": type.googleapis.com/google.protobuf.StringValue
                value: |
                  {}
              vm_config:
                runtime: "envoy.wasm.runtime.v8"
                code:
                  local:
                    filename: "/etc/envoy_filter_http_wasm_example.wasm"
```
3. Istio Envoy 설정 정보
- configPatches
  - context
    - ```
      The specific config generation context to match on. Istio Pilot generates envoy configuration in the context of a gateway, inbound traffic to sidecar and outbound traffic from sidecar.    ```
      ``
  - match : Match on listener/route configuration/cluster.
    - ```
      Conditions specified in a listener match must be met for the patch to be applied to a specific listener across all filter chains, or a specific filter chain inside the listener.
      ```
      - filterChain : Match a specific filter chain in a listener. If specified, the patch will be applied to the filter chain (and a specific filter if specified) and not to other filter chains in the listener.
  - patch : The patch to apply along with the operation
    - operation : 패치가 어떻게 적용될 지를 결정
  - value 
    - The JSON config of the object being patched. This will be merged using proto merge semantics with the existing proto in the path.
  - filterClass
    - Determines the filter insertion order.
  - Response gzip 이 skip 될 수 있는 조건들
    - A request does NOT contain accept-encoding header.
    - A request includes accept-encoding header, but it does not contain “gzip” or “*”.
    - A request includes accept-encoding with “gzip” or “*” with the weight “q=0”. Note that the “gzip” will have a higher weight then “*”. For example, if accept-encoding is “gzip;q=0,*;q=1”, the filter will not compress. But if the header is set to “*;q=0,gzip;q=1”, the filter will compress.
    - A request whose accept-encoding header includes any encoding type with a higher weight than “gzip“‘s given the corresponding compression filter is present in the chain.
    - A response contains a content-encoding header.
    - A response contains a cache-control header whose value includes “no-transform”.
    - A response contains a transfer-encoding header whose value includes a known compression name.
    - A response does not contain a content-type value that matches one of the selected mime-types, which default to application/javascript, application/json, application/xhtml+xml, image/svg+xml, text/css, text/html, text/plain, text/xml.
    - Neither content-length nor transfer-encoding headers are present in the response.
    - Response size is smaller than 30 bytes (only applicable when transfer-encoding is not chunked).
  
### Result
```
resource "kubernetes_manifest" "gzip_envoy_filter" {
  manifest = {
    apiVersion = "networking.istio.io/v1alpha3"
    kind       = "EnvoyFilter"
    metadata   = {
      name      = "gzip"
      namespace = "istio-system"
    }

    spec = {
      configPatches = [
        {
          applyTo = "HTTP_FILTER"
          match = {
            context = "SIDECAR_INBOUND"
            listener = {
              filterChain = {
                filter = {
                  name = "envoy.filters.network.http_connection_manager"
                  subFilter = {
                    name = "envoy.filters.http.router"
                  }
                }
              }
            }
          }
          patch = {
            operation = "INSERT_BEFORE"
            value = {
              name = "envoy.filters.http.compressor"
              typed_config = {
                "@type" = "type.googleapis.com/envoy.extensions.filters.http.compressor.v3.Compressor"
                compressor_library = {
                  name = "text_optimized"
                  typed_config = {
                    "@type" = "type.googleapis.com/envoy.extensions.compression.gzip.compressor.v3.Gzip"
                    compression_level = "BEST_COMPRESSION"
                    compression_strategy = "DEFAULT_STRATEGY"
                  }
                }
              }
            }
          }
        }
      ]
    }
  }
}
```

### Reference
- [Envoy Gzip](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/http/gzip/v3/gzip.proto)
- [Envoy Deprecated Name](https://www.envoyproxy.io/docs/envoy/latest/version_history/v1.14/v1.14.0.html#deprecated)
- [Envoy Filter](https://istio.io/latest/docs/reference/config/networking/envoy-filter/)
- [Activate gzip compression on the Istio-proxy workload](https://gokhan-karadas1992.medium.com/activate-gzip-compression-on-the-istio-proxy-workload-b75f20ea257)
- [Envoy 1.25.0 Compressor](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/compressor_filter)