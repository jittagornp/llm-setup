# LLM Setup

### Article

- [ทำ Authentication ให้ LLM API ด้วย APISIX](https://medium.com/lotuss-it/%E0%B8%97%E0%B8%B3-authentication-%E0%B9%83%E0%B8%AB%E0%B9%89-llm-api-%E0%B8%94%E0%B9%89%E0%B8%A7%E0%B8%A2-apisix-1cda2a6aaba8)
- [วิธีการ Setup Https ให้กับ APISIX](https://medium.com/lotuss-it/%E0%B8%A7%E0%B8%B4%E0%B8%98%E0%B8%B5%E0%B8%81%E0%B8%B2%E0%B8%A3-setup-https-%E0%B9%83%E0%B8%AB%E0%B9%89%E0%B8%81%E0%B8%B1%E0%B8%9A-apisix-8a654f32c1a3)


### Install APISIX
```
cd apisix
docker compose up -d
```

### Run LLM Model

```sh
docker run -d --runtime nvidia --gpus all \
-p 4001:30000 \
-v /data/models/.cache/huggingface:/root/.cache/huggingface \
-e "HF_TOKEN=<YOUR_HUGGING_FACE_TOKEN>" \
--ipc=host \
--network ai \
--restart always \
--name sglang-neuralmagic-Meta-Llama-3.1-70B-Instruct-FP8 \
lmsysorg/sglang:latest \
python3 -m sglang.launch_server \
--model-path neuralmagic/Meta-Llama-3.1-70B-Instruct-FP8 \
--host 0.0.0.0 --port 30000 \
--mem-fraction-static 0.5 \
--tp 2
```

### CURL LLM API via APISIX

```sh
curl http://localhost/llm/neuralmagic/Meta-Llama-3.1-70B-Instruct-FP8/v1/completions \
-H "Content-Type: application/json" \
-d '{
     "model": "neuralmagic/Meta-Llama-3.1-70B-Instruct-FP8",
     "prompt": "What is the LLM?",
     "temperature": 0.7
   }' -H "Authorization: Bearer 1234"
```

### Custom SGLang Docker image

Dockerfile
```
FROM lmsysorg/sglang:latest

# Install python-multipart
RUN pip install python-multipart
```

Build image
```sh
docker build -t lotuss/sglang .
```
