# Google Tag Gateway (GTG) - Cloudflare Worker

Este projeto é um exemplo de implementação do [Google Tag Gateway for Advertisers](https://developers.google.com/tag-platform/tag-manager/gateway/setup-guide?setup=manual#other) utilizando a tecnologia [Cloudflare Workers](https://developers.cloudflare.com/workers/).

## Arquitetura e Fluxo

Para entender melhor como as requisições são roteadas, manipuladas e cacheadas, confira o diagrama de arquitetura:

![Diagrama de Arquitetura e Fluxo](./flow-diagram.png)

## Problema de Cache do Proxy GTG e a Variável `cf`

O cabeçalho de resposta `Cache-Control` dos scripts GTG e sGTM impede o cache na CDN ao retornar `private, max-age=900` em vez de `public, max-age=900`. Mais informações [aqui](./GTG%20cache-control.md).

### Como isso foi resolvido?

Para modificar o comportamento default da Cloudflare e habilitar o cache na CDN, sobrescrevemos as propriedades `cacheTtl` e `cacheEverything` do objeto [RequestInitCfProperties](https://developers.cloudflare.com/workers/runtime-apis/request/#the-cf-property-requestinitcfproperties) utilizado na sub-request feita dentro do Worker. Ver [resolveEdgeCacheOptions](src/index.ts#L55-L61).

## Deploy do Cloudflare Worker

Certifique-se de estar logado na conta Cloudflare correta: `npx wrangler login`

Algumas configurações do Worker ficam no arquivo wrangler.jsonc. Toda vez que ele for modificado é necessário executar `npx wrangler types`.

Finalmente, para efetuar deploy: `npx wrangler deploy`.

## Teste pós-deploy

Após o deplaoy, as rotas `cloudflare-worker.lcrespilh.us/gtg/*` e `cloudflare-worker.lcrespilh.us/sgtm/*` estarão sob controle do Cloudflare Worker.
A maneira mais fácil de testar é abrindo esta [página de teste](https://cloudflare-worker.lcrespilh.us/gtg-cloudflare-worker.html?server_container_url=https://cloudflare-worker.lcrespilh.us/sgtm&disable_media).

Caso ela esteja fora do ar, uma alternativa é usar o curl. Exemplo:

```sh
curl -s -o /dev/null https://cloudflare-worker.lcrespilh.us/gtg/ -w "%header{cache-control} | %header{cf-cache-status}\n"
```

Saída esperada: `public, max-age=900 | HIT`
