# Pre-train Mistral-7B-v0.1 on dolphin dataset using Nemo Meagtron-LM

This example shows how to do parameter efficient fine tuning (PEFT) of [Mistral-7B-v0.1](https://huggingface.co/mistralai/Mistral-7B-v0.1/commits/main) model on [dolphin](https://huggingface.co/datasets/cognitivecomputations/dolphin) dataset using [Nemo](https://github.com/NVIDIA/NeMo) [Megatron-LM](https://github.com/NVIDIA/Megatron-LM).  

## Prerequisites

Before proceeding, complete the [Prerequisites](../../../../README.md#prerequisites) and [Getting started](../../../../README.md#getting-started). 

See [What is in the YAML file](../../../../README.md#yaml-recipes) to understand the common fields in the Helm values files. There are some fields that are specific to a machine learning chart.


## Implicitly defined environment variables

Following variables are implicitly defined by the [pytorch-distributed](../../../charts/machine-learning/training/pytorchjob-distributed/Chart.yaml) Helm chart for use with [Torch distributed run](https://github.com/pytorch/pytorch/blob/main/torch/distributed/run.py):

1. `PET_NNODES` : Maps to `nnodes`
2. `PET_NPROC_PER_NODE` : Maps to `nproc_per_node` 
3. `PET_NODE_RANK` : Maps to `node_rank` 
4. `PET_MASTER_ADDR`: Maps to `master_addr` 
5. `PET_MASTER_PORT`: Maps to `master_port`

## Build and Push Docker Container

This example uses a custom Docker container for [Nemo](https://github.com/NVIDIA/NeMo.git) [Megatron-LM](https://github.com/NVIDIA/Megatron-LM.git). Build and push this container using following command (replace `aws-region` with your AWS Region name):

     cd ~/amazon-eks-machine-learning-with-terraform-and-kubeflow
     ./containers/nemo-megatron/build_tools/build_and_push.sh aws-region


## Hugging Face Mistral 7B pre-trained model weights

To download Hugging Face Mistral 7B pre-trained model weights, replace `YourHuggingFaceToken` with your Hugging Face token below, and execute:

cd ~/amazon-eks-machine-learning-with-terraform-and-kubeflow
helm install --debug nemo-mistral-7b-v01-peft-dolphin     \
    charts/machine-learning/model-prep/hf-snapshot    \
    --set-json='env=[{"name":"HF_MODEL_ID","value":"mistralai/Mistral-7B-v0.1"},{"name":"HF_TOKEN","value":"YourHuggingFaceToken"}]' \
    -n kubeflow-user-example-com

Uninstall the Helm chart at completion:

    helm uninstall nemo-mistral-7b-v01-peft-dolphin -n kubeflow-user-example-com

## Convert HuggingFace Checkpoint to Nemo Checkpoint

To convert checkpoint:

    cd ~/amazon-eks-machine-learning-with-terraform-and-kubeflow
    helm install --debug nemo-mistral-7b-v01-peft-dolphin \
        charts/machine-learning/data-prep/data-process \
        -f examples/training/nemo-megatron/mistral-7b-v01-peft-dolphin/hf_to_nemo.yaml -n kubeflow-user-example-com

To monitor the logs, execute:

    kubectl logs -f data-process-nemo-mistral-7b-v01-peft-dolphin -n kubeflow-user-example-com

Uninstall the Helm chart at completion:

    helm uninstall nemo-mistral-7b-v01-peft-dolphin -n kubeflow-user-example-com

## Preprocess `dolphin`` dataset

To preprocess `dolphin` dataset:

    cd ~/amazon-eks-machine-learning-with-terraform-and-kubeflow
    helm install --debug nemo-mistral-7b-v01-peft-dolphin \
        charts/machine-learning/data-prep/data-process \
        -f examples/training/nemo-megatron/mistral-7b-v01-peft-dolphin/preprocess.yaml -n kubeflow-user-example-com

To monitor the logs, execute:

    kubectl logs -f data-process-nemo-mistral-7b-v01-peft-dolphin -n kubeflow-user-example-com

Uninstall the Helm chart at completion:

    helm uninstall nemo-mistral-7b-v01-peft-dolphin -n kubeflow-user-example-com

## Peft

To do peft:

    cd ~/amazon-eks-machine-learning-with-terraform-and-kubeflow
    helm install --debug nemo-mistral-7b-v01-peft-dolphin \
       charts/machine-learning/training/pytorchjob-distributed \
        -f examples/training/nemo-megatron/mistral-7b-v01-peft-dolphin/peft.yaml -n kubeflow-user-example-com

To monitor the logs, execute:

    kubectl logs -f data-process-nemo-mistral-7b-v01-peft-dolphin -n kubeflow-user-example-com

Uninstall the Helm chart at completion:

    helm uninstall nemo-mistral-7b-v01-peft-dolphin -n kubeflow-user-example-com

## Evaluate 

To evaluate peft trained model over test dataset:

    cd ~/amazon-eks-machine-learning-with-terraform-and-kubeflow
    helm install --debug nemo-mistral-7b-v01-peft-dolphin \
       charts/machine-learning/training/pytorchjob-distributed \
        -f examples/training/nemo-megatron/mistral-7b-v01-peft-dolphin/peft_eval.yaml -n kubeflow-user-example-com

To monitor the logs, execute:

    kubectl logs -f data-process-nemo-mistral-7b-v01-peft-dolphin -n kubeflow-user-example-com

Uninstall the Helm chart at completion:

    helm uninstall nemo-mistral-7b-v01-peft-dolphin -n kubeflow-user-example-com


## Merge PEFT Model to Base Model

To merge PEFT model to base model:

    cd ~/amazon-eks-machine-learning-with-terraform-and-kubeflow
    helm install --debug nemo-mistral-7b-v01-peft-dolphin \
        charts/machine-learning/data-prep/data-process \
        -f examples/training/nemo-megatron/mistral-7b-v01-peft-dolphin/merge_peft.yaml -n kubeflow-user-example-com

To monitor the logs, execute:

    kubectl logs -f data-process-mistral-7b-v01-peft-dolphin -n kubeflow-user-example-com

Uninstall the Helm chart at completion:

    helm uninstall nemo-mistral-7b-v01-peft-dolphin -n kubeflow-user-example-com

## Convert Nemo Checkpoint to Hugging Face Checkpoint

To convert checkpoint:

    cd ~/amazon-eks-machine-learning-with-terraform-and-kubeflow
    helm install --debug nemo-mistral-7b-v01-peft-dolphin \
        charts/machine-learning/data-prep/data-process \
        -f examples/training/nemo-megatron/mistral-7b-v01-peft-dolphin/nemo_to_hf.yaml -n kubeflow-user-example-com

To monitor the logs, execute:

    kubectl logs -f data-process-nemo-mistral-7b-v01-peft-dolphin -n kubeflow-user-example-com

Uninstall the Helm chart at completion:

    helm uninstall nemo-mistral-7b-v01-peft-dolphin -n kubeflow-user-example-com

## Output

To access the output stored on EFS and FSx for Lustre file-systems, execute following commands:

    cd ~/amazon-eks-machine-learning-with-terraform-and-kubeflow
    kubectl apply -f eks-cluster/utils/attach-pvc.yaml  -n kubeflow
    kubectl exec -it -n kubeflow attach-pvc -- /bin/bash


This will put you in a pod attached to the  EFS and FSx for Lustre file-systems, mounted at `/efs`, and `/fsx`, respectively. Type `exit` to exit the pod.

### Data

Pre-processed data is available in `/fsx/home/nemo-mistral-7b-v01-peft-dolphin/data/` folder.

### Logs

Pre-training `logs` are available in `/efs/home/nemo-mistral-7b-v01-peft-dolphin/logs` folder. 

### S3 Backup

Any content stored under `/fsx` is automatically backed up to your configured S3 bucket.