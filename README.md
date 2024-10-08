# uni-api

<p align="center">
  <a href="https://t.me/uni_api">
    <img src="https://img.shields.io/badge/Join Telegram Group-blue?&logo=telegram">
  </a>
   <a href="https://hub.docker.com/repository/docker/yym68686/uni-api">
    <img src="https://img.shields.io/docker/pulls/yym68686/uni-api?color=blue" alt="docker pull">
  </a>
</p>

[English](./README.md) | [Chinese](./README_CN.md)

## Introduction

If used personally, one/new-api is too complex and has many commercial features that individuals do not need. If you do not want a complex front-end interface and want to support more models, you can try uni-api. This is a project that unifies the management of large model APIs, allowing multiple backend services to be called through a unified API interface and uniformly converted to the OpenAI format, supporting load balancing. Currently supported backend services include: OpenAI, Anthropic, Gemini, Vertex, Cohere, Groq, Cloudflare, DeepBricks, OpenRouter, etc.

## Features

- No frontend, pure configuration file setup for API channels. You can run your own API site by just writing one file, with detailed configuration guides in the documentation, beginner-friendly.
- Unified management of multiple backend services, supporting providers like OpenAI, Deepseek, DeepBricks, OpenRouter, and other APIs in the OpenAI format. Supports OpenAI Dalle-3 image generation.
- Supports Anthropic, Gemini, Vertex AI, Cohere, Groq, Cloudflare. Vertex supports both Claude and Gemini APIs.
- Supports OpenAI, Anthropic, Gemini, Vertex native tool use function calls.
- Supports OpenAI, Anthropic, Gemini, Vertex native image recognition API.
- Supports four types of load balancing.
  1. Supports channel-level weighted load balancing, which can allocate requests based on different channel weights. Disabled by default, requires channel weight configuration.
  2. Supports Vertex regional load balancing, supports Vertex high concurrency, and can increase Gemini, Claude concurrency by up to (number of APIs * number of regions) times. Automatically enabled without additional configuration.
  3. In addition to Vertex region-level load balancing, all APIs support channel-level sequential load balancing, enhancing the immersive translation experience. Automatically enabled without additional configuration.
  4. Support automatic API key-level round-robin load balancing for multiple API Keys in a single channel.
- Supports automatic retry, when an API channel response fails, automatically retry the next API channel.
- Supports fine-grained access control. Supports using wildcards to set specific models for API key available channels.
- Supports rate limiting, can set the maximum number of requests per minute, can be set as an integer, such as 2/min, 2 times per minute, 5/hour, 5 times per hour, 10/day, 10 times per day, 10/month, 10 times per month, 10/year, 10 times per year. Default is 60/min.
- Supports multiple standard OpenAI format interfaces: `/v1/chat/completions`, `/v1/images/generations`, `/v1/audio/transcriptions`, `/v1/moderations`, `/v1/models`.
- Supports OpenAI moderation for ethical review, allowing for ethical review of user messages. If inappropriate messages are detected, an error message will be returned. This reduces the risk of the backend API being banned by providers.

## Configuration

Using the api.yaml configuration file, multiple models can be configured, and each model can be configured with multiple backend services, supporting load balancing. Below is an example of the api.yaml configuration file:

```yaml
providers:
  - provider: provider_name # Service provider name, such as openai, anthropic, gemini, openrouter, deepbricks, any name is fine, required
    base_url: https://api.your.com/v1/chat/completions # Backend service API address, required
    api: sk-YgS6GTi0b4bEabc4C # Provider's API Key, required
    model: # At least one model must be filled in
      - gpt-4o # Usable model name, required
      - claude-3-5-sonnet-20240620: claude-3-5-sonnet # Rename model, claude-3-5-sonnet-20240620 is the provider's model name, claude-3-5-sonnet is the renamed name, you can use a concise name instead of the original complex name, optional
      - dall-e-3

  - provider: anthropic
    base_url: https://api.anthropic.com/v1/messages
    api: # Supports multiple API Keys, multiple keys automatically enable polling load balancing, at least one key, required
      - sk-ant-api03-bNnAOJyA-xQw_twAA
      - sk-ant-api02-bNnxxxx
    model:
      - claude-3-5-sonnet-20240620: claude-3-5-sonnet # Rename model, claude-3-5-sonnet-20240620 is the provider's model name, claude-3-5-sonnet is the renamed name, you can use a concise name instead of the original complex name, optional
    tools: true # Whether to support tools, such as generating code, generating documents, etc., default is true, optional

  - provider: gemini
    base_url: https://generativelanguage.googleapis.com/v1beta # base_url supports v1beta/v1, only for Gemini models, required
    api: AIzaSyAN2k6IRdgw
    model:
      - gemini-1.5-pro
      - gemini-1.5-flash-exp-0827: gemini-1.5-flash # After renaming, the original model name gemini-1.5-flash-exp-0827 cannot be used. If you want to use the original name, you can add the original name in the model, just add the following line to use the original name.
      - gemini-1.5-flash-exp-0827 # Add this line, both gemini-1.5-flash-exp-0827 and gemini-1.5-flash can be requested
    tools: true

  - provider: vertex
    project_id: gen-lang-client-xxxxxxxxxxxxxx #    Description: Your Google Cloud project ID. Format: String, usually composed of lowercase letters, numbers, and hyphens. How to obtain: You can find your project ID in the project selector of the Google Cloud Console.
    private_key: "-----BEGIN PRIVATE KEY-----\nxxxxx\n-----END PRIVATE" # Description: Private key of the Google Cloud Vertex AI service account. Format: A JSON formatted string containing the private key information of the service account. How to obtain: Create a service account in the Google Cloud Console, generate a JSON formatted key file, and then set its content as the value of this environment variable.
    client_email: xxxxxxxxxx@xxxxxxx.gserviceaccount.com # Description: Email address of the Google Cloud Vertex AI service account. Format: Usually a string like "service-account-name@project-id.iam.gserviceaccount.com". How to obtain: Generated when creating the service account, you can also view the service account details in the "IAM & Admin" section of the Google Cloud Console.
    model:
      - gemini-1.5-pro
      - gemini-1.5-flash
      - claude-3-5-sonnet@20240620: claude-3-5-sonnet
      - claude-3-opus@20240229: claude-3-opus
      - claude-3-sonnet@20240229: claude-3-sonnet
      - claude-3-haiku@20240307: claude-3-haiku
    tools: true
    notes: https://xxxxx.com/ # You can put the provider's website, notes, official documentation, optional

  - provider: cloudflare
    api: f42b3xxxxxxxxxxq4aoGAh # Cloudflare API Key, required
    cf_account_id: 8ec0xxxxxxxxxxxxe721 # Cloudflare Account ID, required
    model:
      - '@cf/meta/llama-3.1-8b-instruct': llama-3.1-8b # Rename model, @cf/meta/llama-3.1-8b-instruct is the provider's original model name, the model name must be enclosed in quotes, otherwise yaml syntax error, llama-3.1-8b is the renamed name, you can use a concise name instead of the original complex name, optional
      - '@cf/meta/llama-3.1-8b-instruct' # The model name must be enclosed in quotes, otherwise yaml syntax error

  - provider: other-provider
    base_url: https://api.xxx.com/v1/messages
    api: sk-bNnAOJyA-xQw_twAA
    model:
      - causallm-35b-beta2ep-q6k: causallm-35b
      - anthropic/claude-3-5-sonnet
    tools: false
    engine: openrouter # Force to use a specific message format, currently supports gpt, claude, gemini, openrouter native format, optional

api_keys:
  - api: sk-KjjI60Yf0JFWxfgRmXqFWyGtWUd9GZnmi3KlvowmRWpWpQRo # API Key, users need an API key to use this service, required
    model: # Models that this API Key can use, required
      - gpt-4o # Usable model name, can use all gpt-4o models provided by providers
      - claude-3-5-sonnet # Usable model name, can use all claude-3-5-sonnet models provided by providers
      - gemini/* # Usable model name, can only use all models provided by the provider named gemini, where gemini is the provider name, * represents all models
    role: admin

  - api: sk-pkhf60Yf0JGyJxgRmXqFQyTgWUd9GZnmi3KlvowmRWpWqrhy
    model:
      - anthropic/claude-3-5-sonnet # Usable model name, can only use the claude-3-5-sonnet model provided by the provider named anthropic. Other providers' claude-3-5-sonnet models cannot be used. This way of writing will not match the model named anthropic/claude-3-5-sonnet provided by other-provider.
      - <anthropic/claude-3-5-sonnet> # By adding angle brackets on both sides of the model name, it will not look for the claude-3-5-sonnet model under the channel named anthropic, but will treat the entire anthropic/claude-3-5-sonnet as the model name. This way of writing can match the model named anthropic/claude-3-5-sonnet provided by other-provider. But it will not match the claude-3-5-sonnet model under anthropic.
      - openai-test/text-moderation-latest # When message moderation is enabled, you can use the text-moderation-latest model under the channel named openai-test for moderation.
    preferences:
      USE_ROUND_ROBIN: true # Whether to use polling load balancing, true to use, false to not use, default is true. When polling is enabled, each request will be made in the order configured in the model. It is not related to the original channel order in providers. Therefore, you can set different request orders for each API key.
      AUTO_RETRY: true # Whether to automatically retry, automatically retry the next provider, true to automatically retry, false to not automatically retry, default is true
      RATE_LIMIT: 2/min # Supports rate limiting, the maximum number of requests per minute, can be set to an integer, such as 2/min, 2 times per minute, 5/hour, 5 times per hour, 10/day, 10 times per day, 10/month, 10 times per month, 10/year, 10 times per year. Default is 60/min, optional
      ENABLE_MODERATION: true # Whether to enable message moderation, true to enable, false to not enable, default is false. When enabled, it will conduct moderation on the user's message, if inappropriate messages are found, it will return an error message.

  # Channel-level weighted load balancing configuration example
  - api: sk-KjjI60Yd0JFWtxxxxxxxxxxxxxxwmRWpWpQRo
    model:
      - gcp1/*: 5 # The number after the colon is the weight, the weight only supports positive integers.
      - gcp2/*: 3 # The larger the number, the greater the probability of the request.
      - gcp3/*: 2 # In this example, there are a total of 10 weights for all channels, and 5 out of 10 requests will request the gcp1/* model, 2 requests will request the gcp2/* model, and 3 requests will request the gcp3/* model.

    preferences:
      USE_ROUND_ROBIN: true # When USE_ROUND_ROBIN must be true and there is no weight after the above channels, it will request in the original channel order, if there is weight, it will request in the weighted order.
      AUTO_RETRY: true
```

If you do not want to set available channels for each `api` one by one in `api_keys`, `uni-api` supports setting the `api key` to be able to use all models. The configuration is as follows:

```yaml
# ... providers configuration unchanged ...
api_keys:
  - api: sk-LjjI60Yf0JFWxfgRmXqFWyGtWUd9GZnmi3KlvowmRWpWpQRo # API Key, users need an API key to request uni-api, required
    model: # The model that can be used with this API Key, required
      - all # Can use all models in all channels set under providers, no need to add available channels one by one.
# ... other configurations unchanged ...
```

## Environment Variables

- CONFIG_URL: The download address of the configuration file, it can be a local file or a remote file, optional
- TIMEOUT: Request timeout, default is 100 seconds, the timeout can control the time needed to switch to the next channel when a channel does not respond. Optional

## Retrieve Statistical Data

Use `/stats` to get usage statistics for each channel over the last 24 hours. Include your own uni-api admin API key.

The data includes:

1. Success rate for each model under each channel, sorted from highest to lowest success rate.
2. Overall success rate for each channel, sorted from highest to lowest.
3. Total number of requests for each model across all channels.
4. Number of requests for each endpoint.
5. Number of requests from each IP address.

`/stats?hours=48` The `hours` parameter can control how many hours of recent data statistics are returned. If the `hours` parameter is not provided, it defaults to statistics for the last 24 hours.

There are other statistical data that you can query yourself by writing SQL in the database. Other data includes: first token time, total processing time for each request, whether each request was successful, whether each request passed ethical review, text content of each request, API key for each request, input token count, and output token count for each request.

## Docker Local Deployment

Start the container

```bash
docker run --user root -p 8001:8000 --name uni-api -dit \
-e CONFIG_URL=http://file_url/api.yaml \ # If the local configuration file is already mounted, you do not need to set CONFIG_URL
-v ./api.yaml:/home/api.yaml \ # If CONFIG_URL is already set, you do not need to mount the configuration file
-v ./uniapi_db:/home/data \ # If you do not want to save statistical data, you do not need to mount the stats.db file
yym68686/uni-api:latest
```

Or if you want to use Docker Compose, here is a docker-compose.yml example:

```yaml
services:
  uni-api:
    container_name: uni-api
    image: yym68686/uni-api:latest
    environment:
      - CONFIG_URL=http://file_url/api.yaml # If the local configuration file is already mounted, there is no need to set CONFIG_URL
    ports:
      - 8001:8000
    volumes:
      - ./api.yaml:/home/api.yaml # If CONFIG_URL is already set, there is no need to mount the configuration file
      - ./uniapi_db:/home/data # If you do not want to save statistical data, there is no need to mount the stats.db file
```

CONFIG_URL is used to automatically download remote configuration files. For example, if it is inconvenient to modify the configuration file on a certain platform, you can upload the configuration file to a hosting service and provide a direct link for uni-api to download. CONFIG_URL is this direct link. If you are using a locally mounted configuration file, you do not need to set CONFIG_URL. CONFIG_URL is used in situations where it is inconvenient to mount the configuration file.

Run Docker Compose container in the background

```bash
docker-compose pull
docker-compose up -d
```

Docker build

```bash
docker build --no-cache -t uni-api:latest -f Dockerfile --platform linux/amd64 .
docker tag uni-api:latest yym68686/uni-api:latest
docker push yym68686/uni-api:latest
```

One-Click Restart Docker Image

```bash
set -eu
docker pull yym68686/uni-api:latest
docker rm -f uni-api
docker run --user root -p 8001:8000 -dit --name uni-api \
-e CONFIG_URL=http://file_url/api.yaml \
-v ./api.yaml:/home/api.yaml \
-v ./uniapi_db:/home/data \
yym68686/uni-api:latest
docker logs -f uni-api
```

RESTful curl test

```bash
curl -X POST http://127.0.0.1:8000/v1/chat/completions \
-H "Content-Type: application/json" \
-H "Authorization: Bearer ${API}" \
-d '{"model": "gpt-4o","messages": [{"role": "user", "content": "Hello"}],"stream": true}'
```


## Star History

<a href="https://github.com/yym68686/uni-api/stargazers">
        <img width="500" alt="Star History Chart" src="https://api.star-history.com/svg?repos=yym68686/uni-api&type=Date">
</a>