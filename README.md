# Documentação Técnica: Projeto DevSecOps com Rancher e GitOps

Projeto: GitOps na prática

Versão: 1.0

Data: 09 de Setembro de 2025

------------------

## 1. Visão Geral

Este documento detalha o processo de implantação de uma aplicação de microserviços em um ambiente Kubernetes local, gerenciado através de práticas de GitOps. O objetivo é utilizar um repositório Git como a única fonte para o estado desejado da aplicação e empregar a ferramenta ArgoCD para automatizar a sincronização desse estado com o cluster Kubernetes.

A solução demonstra um fluxo de trabalho moderno utilizada no mercado, onde as alterações na configuração da infraestrutura e da aplicação são feitas via commits no Git, garantindo um processo versionado e auditável.

### 1.1. Tecnologias Utilizadas:

[GitHub](https://github.com/): Plataforma de hospedagem do repositório Git, servindo como a fonte da configuração da aplicação.

[Rancher-Desktop](https://rancherdesktop.io/): Containerização e orquestração dos serviços.

[Kubernet](https://kubernetes.io/releases/download/): Plataforma de orquestração de containers, responsável por gerenciar o ciclo de vida da aplicação de microserviços.

[ArgoCD](https://github.com/badtuxx/DescomplicandoArgoCD/blob/main/pt/src/day-1/README.md#instalando-o-argocd): O link do ArgoCD irá encaminhar para o repositório do [badtuxx](https://github.com/badtuxx/). Siga as instruções de instalação dele. O ArgoCD é uma ferramenta de GitOps para Kubernetes, utilizada para automatizar o deploy e a sincronização da aplicação a partir do repositório Git.

------------------

## 2. Arquitetura da Solução (Versão Corrigida)

A arquitetura se baseia em serviços do em um fluxo de trabalho GitOps que conecta três componentes principais:

### 2.1. Repositório Git (GitHub):

 Atua como a fonte da verdade (source of truth).

 Armazena os arquivos YAML do Kubernetes que descrevem todos os recursos da aplicação Online Boutique (Deployments, Services, etc.).

 Toda alteração no estado desejado da aplicação deve ser feita através de um commit neste repositório.

    
### 2.2. ArgoCD (Ferramenta de GitOps):

 Atua como o controlador que automatiza o deploy.

 Fica em execução dentro do cluster Kubernetes e monitora continuamente o repositório Git.

 Ele compara o estado definido nos manifestos do Git (estado desejado) com o estado dos recursos que estão realmente em execução no cluster (estado atual).

 Se houver uma divergência, o ArgoCD sincroniza automaticamente ou manualmente as mudanças necessárias no cluster para garantir que o estado atual corresponda ao estado desejado.

### 2.3. Cluster Kubernetes (Rancher Desktop):

 É o ambiente de execução onde a aplicação é implantada.

 Orquestra os containers dos múltiplos microserviços que compõem a aplicação Online Boutique.

 Recebe as instruções declarativas (aplicação dos manifestos) do ArgoCD.

#### Fluxo de Trabalho:
Desenvolvedor → git push no repositório GitHub → ArgoCD detecta a mudança → ArgoCD aplica o manifesto no Cluster Kubernetes → Aplicação é atualizada.

## 3. Guia de Execução Passo a Passo

Esta seção descreve as etapas para implantar a aplicação Online Boutique utilizando o fluxo de GitOps.

> **Nota:**
> Esta documentação não inclui tutorial de completo de instalação de todos os componentes. No item [Tecnologias Utilizadas](### 1.1. Tecnologias Utilizadas:). Estamos partindo do pressuposto que já é usuário do GitHub e possui o comando git instalado e logado no seu sistema operacional.


### 3.1. Preparação do Repositório Git


Crie um repositorio novo na sua conta GitHub e salve as instruções que são geradas na última página

![GitHub instruções](/imgs/GitHub%20-%20primeiros%20comandos.png)

Dentro da pasta no SO que irá ser upado seu repositório, crie as pastas nesta sequencia:

           [Seu Repositório]/gitops-microservices/k8s/

E coloque o arquivo [online-boutique.yaml](/gitops-microservices/k8s/online-boutique.yaml)

Dê o commit corretamente e salve o link do seu repositório para ser usado à frente.

### 3.2. Instalação do ArgoCD no Cluster

No terminal, crie um namespace para a instalação do ArgoCD

            kubectl create namespace argocd

Abaixo, aplique o link da instalação 

            kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

Verificação: Aguarde alguns minutos e verifique se os pods do ArgoCD estão em execução:


            kubectl get pods -n argocd
