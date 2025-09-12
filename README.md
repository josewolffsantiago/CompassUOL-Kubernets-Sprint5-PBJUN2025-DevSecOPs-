# Documentação Técnica: Projeto DevSecOps com Rancher e GitOps

Projeto: GitOps na prática

Versão: 1.1

Data: 11 de Setembro de 2025


## Sumário

1. [Visão Geral](#1-visão-geral)
   - [1.1. Tecnologias Utilizadas](#11-tecnologias-utilizadas)
2. [Arquitetura da Solução](#2-arquitetura-da-solução-versão-corrigida)
   - [2.1. Repositório Git (GitHub)](#21-repositório-git-github)
   - [2.2. ArgoCD (Ferramenta de GitOps)](#22-argocd-ferramenta-de-gitops)
   - [2.3. Cluster Kubernetes (Rancher Desktop)](#23-cluster-kubernetes-rancher-desktop)
   - [Fluxo de Trabalho](#fluxo-de-trabalho)
3. [Guia de Execução Passo a Passo](#3-guia-de-execução-passo-a-passo)
   - [3.1. Preparação do Repositório Git](#31-preparação-do-repositório-git)
   - [3.2. Instalação do ArgoCD no Cluster](#32-instalação-do-argocd-no-cluster)
   - [3.3. Acesso à Interface Web do ArgoCD](#33-acesso-à-interface-web-do-argocd)
   - [3.4. Criação da Aplicação no ArgoCD](#34-criação-da-aplicação-no-argocd)
   - [3.5. Sincronização e Acesso à Aplicação](#35-sincronização-e-acesso-à-aplicação)
4. [Implementação do MetalLB](#4-implantação-do-metallb)
   - [4.1. Instalação do MetalLB](#41-instalação-do-metallb)
   - [4.2. Configuração do MetalLB](#42-configuração-do-metallb)
   - [4.3. Resultado](#43-resultado)
5. [Referências](#5-referências)



------------------

## 1. Visão Geral

Este documento detalha o processo de implantação de uma aplicação de microserviços em um ambiente Kubernetes local, gerenciado através de práticas de GitOps. O objetivo é utilizar um repositório Git como a única fonte para o estado desejado da aplicação e empregar a ferramenta ArgoCD para automatizar a sincronização desse estado com o cluster Kubernetes.

A solução demonstra um fluxo de trabalho moderno utilizada no mercado, onde as alterações na configuração da infraestrutura e da aplicação são feitas via commits no Git, garantindo um processo versionado e auditável.

### 1.1. Tecnologias Utilizadas:

[GitHub](https://github.com/): Plataforma de hospedagem do repositório Git, servindo como a fonte da configuração da aplicação.

[Rancher-Desktop](https://rancherdesktop.io/): Containerização e orquestração dos serviços.

[Kubernetes](https://kubernetes.io/releases/download/): Plataforma de orquestração de containers, responsável por gerenciar o ciclo de vida da aplicação de microserviços.

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

------------------

## 3. Guia de Execução Passo a Passo

Esta seção descreve as etapas para implantar a aplicação Online Boutique utilizando o fluxo de GitOps.

>[!NOTE]
>Esta documentação não inclui tutorial de completo de instalação de todos os componentes. No item [Tecnologias Utilizadas](https://github.com/josewolffsantiago/CompassUOL-Kubernets-Sprint5-PBJUN2025-DevSecOPs-?tab=readme-ov-file#11-tecnologias-utilizadas) tem todos os links para a instalação e configuração correta de cada componente para a Execução deste projeto.


### 3.1. Preparação do Repositório Git


Crie um repositorio novo na sua conta GitHub e salve as instruções que são geradas na última página

![GitHub instruções](/imgs/GitHub%20-%20primeiros%20comandos.png)

Dentro da pasta no SO que irá ser upado seu repositório, crie as pastas nesta sequencia:

           [Seu Repositório]/gitops-microservices/k8s/

E coloque o arquivo [online-boutique.yaml](/gitops-microservices/k8s/online-boutique.yaml)

Dê o commit corretamente e salve o link do seu repositório para ser usado à frente.

Exemplo de como ficará a pasta local antes de dar o primeiro "commit" no github.

![Manifesto YAML no local](/imgs/Pasta%20localização%20-%20Manifesto%20yaml.png)

>[!IMPORTANT] 
>Veja que o meu "git init" foi realizado dentro da pasta "Kubernetes - ArgoCD"

### 3.2. Instalação do ArgoCD no Cluster

No terminal, crie um namespace para a instalação do ArgoCD

            kubectl create namespace argocd

Abaixo, aplique o link da instalação 

            kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

Verificação: Aguarde alguns minutos e verifique se os pods do ArgoCD estão em execução:


            kubectl get pods -n argocd

### 3.3. Acesso à Interface Web do ArgoCD

Para acessar a interface do ArgoCD, que por padrão não é exposta externamente, execute o comando:

            kubectl port-forward svc/argocd-server -n argocd 8080:443

Obter a Senha Inicial: A senha inicial do usuário admin é gerada automaticamente e armazenada em um Secret. Para obtê-la, execute:

            kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

Login: Acesse https://localhost:8080 em seu navegador. Use admin como usuário e a senha obtida no passo anterior.

### 3.4. Criação da aplicação no ArgoCD

Voltando ao terminal, vamos criar um novo namespace no Kubernetes utilizando o kubectl. O namespace é importante para organizarmos os aplicativos e também para especificarmos os comandos kubectl diretamente para o aplicativo que será necessário alguma alteração.

            kubectl create namespace online-boutique

Logo após, colocamos um comando para o argocd criar diretamente o nosso aplicativo e poder fazer o gerenciamento do mesmo.

            argocd app create online-boutique     --repo [Aqui você vai colocar o link do seu repositório .git]     --path gitops-microservices/k8s/     --dest-server https://kubernetes.default.svc     --dest-namespace online-boutique

O item dest-server está apontando para a criação no cluster local.

### 3.5. Sincronização e Acesso à Aplicação

Após a criação, o ArgoCD precisa iniciar o processo de sincronização (Sync), pois o default dele é não sincronicar automaticamente.

Coloque no terminal este comando:

            argocd app sync online-boutique

Ao acessar o https://localhost:8080, veja que o aplicativo já vai aparecer e aguarde que o status da aplicação mude para Healthy e Synced.

Verifique se todos os pods dos microserviços estão rodando:h

            kubectl get pods -n default

O serviço do frontend é do tipo ClusterIP. Para acessá-lo externamente, faça um port-forward:
Bash

            kubectl port-forward -n online-boutique svc/frontend-external 8081:80

Abra seu navegador e acesse http://localhost:8081 para ver a loja Online Boutique funcionando.

------------------

## 4. Implementação do MetalLB

O [MetalLB](https://metallb.io/) é um controlador de load balancer para clusters Kubernetes em ambientes bare-metal ou locais. Ele permite que serviços do tipo LoadBalancer recebam IPs externos roteáveis, tornando possível acessar aplicações do cluster como se estivessem em provedores cloud, onde o balanceamento acontece automaticamente.

Ele monitora serviços do tipo LoadBalancer e atribui um IP externo definido pelo usuário (por exemplo, faixa de IPs da rede local)

![ArgoCD - Progressing](/imgs/ArgoCD%20-%20FrontEnd%20External%20-%20Progressing.png)

### 4.1. Instalação do MetalLB

Para fazer a instalação do MetalLB, é só seguir o manifesto abaixo. O link irá fazer a instalação sempre da ultima versão. Coloque no terminal:

            kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.15.2/config/manifests/metallb-native.yaml

### 4.2. Configuração do MetalLB

Faça o download destes dois arquivos que já estão prontos:

[ippoladress.yaml](/ippooladdress.yaml)

[l2adv.yaml](/l2adv.yaml)

Ambos estes arquivos estão na págida de configuração da [documentação do MetalLB](https://metallb.io/configuration/). Ele insere um IP diretamente no ClusterIP para que o POD FrontEnd External mude o status de "Progressing" para "Healthy". Isto acontece por causa da falta do IP Externo de um LoadBalancer em Soluções Cloud.

Ao baixar os dois arquivos, abra o terminal na localidade que estão salvos e aplique ambos usando o kubectl

            kubectl apply -f ippooladdress.yaml 


            kubectl apply -f l2adv.yaml 

### 4.3. Resultado

Após a adição do dois manifestos, ao acessar o ArgoCD no https://localhost:8080. Verá que a aplicação irá aparecer como "Healthy", conforme imagem abaixo:

![Arcd - Healthy](/imgs/ArgoCD%20-%20FrontEnd%20External%20-%20Healthy.png)

------------------

## 5. Referências

[Descomplicando o ArgoCD e o GitOps! Com livro grátis e desafio prático!](https://www.youtube.com/watch?v=TDvA2vAQCF8&t)

[Documentação do Kubectl run](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_run/)

[Passo a Passo: Configurando MetalLB para Load Balancing em Kubernetes](https://medium.com/@luan.ads359/passo-a-passo-configurando-metallb-para-load-balancing-em-kubernetes-40213a341282)



