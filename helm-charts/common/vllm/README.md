# vllm

Helm chart for deploying vLLM Inference service.

Refer to [Deploy with Helm Charts](../../README.md) for global guides.

## Installing the Chart

To install the chart, run the following:

Note that you cannot use vllm as the service release name due to [environment variables conflict](https://docs.vllm.ai/en/stable/serving/env_vars.html#environment-variables).

```console
cd GenAIInfra/helm-charts/common/vllm
export MODELDIR=/mnt/opea-models
export MODELNAME="meta-llama/Meta-Llama-3-8B-Instruct"
export HFTOKEN="insert-your-huggingface-token-here"
helm install myvllm . --set global.modelUseHostPath=${MODELDIR} --set LLM_MODEL_ID=${MODELNAME} --set global.HUGGINGFACEHUB_API_TOKEN=${HFTOKEN}
# To deploy on Gaudi enabled kubernetes cluster
# helm install myvllm . --set global.modelUseHostPath=${MODELDIR} --set LLM_MODEL_ID=${MODELNAME} --set global.HUGGINGFACEHUB_API_TOKEN=${HFTOKEN} --values gaudi-values.yaml
# To deploy on AMD ROCm GPU kubernetes cluster
# helm install vllm-rocm . --set global.modelUseHostPath=${MODELDIR} --set LLM_MODEL_ID=${MODELNAME} --set global.HUGGINGFACEHUB_API_TOKEN=${HFTOKEN} --values rocm-values.yaml
```

By default, the vllm service will downloading the "meta-llama/Meta-Llama-3-8B-Instruct".

If you already cached the model locally, you can pass it to container like this example:

MODELDIR=/mnt/opea-models

MODELNAME="facebook/opt-125m"

## Verify

To verify the installation, run the command `kubectl get pod` to make sure all pods are runinng.

Then run the command `kubectl port-forward svc/myvllm 2080:80` to expose the vllm service for access.

Open another terminal and run the following command to verify the service if working:

```console
curl http://localhost:2080/v1/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "meta-llama/Meta-Llama-3-8B-Instruct", "prompt": "What is Deep Learning?", "max_tokens": 32, "temperature": 0}'
```

## Values

| Key                             | Type   | Default                                 | Description                                                                                                                                                                                                            |
| ------------------------------- | ------ | --------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| LLM_MODEL_ID                    | string | `"meta-llama/Meta-Llama-3-8B-Instruct"` | Models id from https://huggingface.co/, or predownloaded model directory                                                                                                                                               |
| global.HUGGINGFACEHUB_API_TOKEN | string | `insert-your-huggingface-token-here`    | Hugging Face API token                                                                                                                                                                                                 |
| global.modelUseHostPath         | string | `""`                                    | Cached models directory, vllm will not download if the model is cached here. The host path "modelUseHostPath" will be mounted to container as /data directory. Set this to null/empty will force it to download model. |
| image.repository                | string | `"opea/vllm"`                           |                                                                                                                                                                                                                        |
| image.tag                       | string | `"latest"`                              |                                                                                                                                                                                                                        |
| autoscaling.enabled             | bool   | `false`                                 | Enable HPA autoscaling for the service deployment based on metrics it provides. See [HPA instructions](../../HPA.md) before enabling!                                                                                  |
| global.monitoring               | bool   | `false`                                 | Enable usage metrics for the service. Required for HPA. See [monitoring instructions](../../monitoring.md) before enabling!                                                                                            |
