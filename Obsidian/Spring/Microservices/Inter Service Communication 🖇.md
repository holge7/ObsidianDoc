To be able to call other microservice from other service, we will make use of a #WebClient Class,
this is part of  "Spring Web Flux"

```
@Configuration
public class WebClientConfig {
	
	@Bean
	public WebClient.Builder webClientBuilder() {
		return WebClient.builder();
	}
	
}
```

To use this class, we can do something like:

```
InventoryResponseDTO[] inventoryResposne = webClientBuilder.build()
		.get()
		.uri("http://localhost:8082/api/inventory/service",
				uriBuilder -> uriBuilder.queryParam("skuCodes", skuCodes).build()
			)
		.retrieve()
		.bodyToMono(InventoryResponseDTO[].class)
		.block();
```

