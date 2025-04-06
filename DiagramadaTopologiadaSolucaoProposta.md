# Diagrama de Topologia – Arquitetura Híbrida AWS + On-Premise

```mermaid
graph TD
    User[Usuário Web] --> ClFront[CloudFront]
    
    ClFront[CloudFront] --> APIGW[API Gateway]

    APIGW --> LB[Load Balancer]
    
    LB --> EC2[Instâncias EC2]
    LB --> ECS[Containers ECS/Fargate]
    LB --> Lambda[Funções Lambda]
    
    EC2 --> Aurora[(Banco Aurora - Nuvem)]
    ECS --> Aurora
    Lambda --> Aurora
    
    EC2 --> OnPremDB1[(Oracle - On-premise)]
    EC2 --> OnPremDB2[(SQL Server - On-premise)]
    
    subgraph On-Premise
        OnPremDB1
        OnPremDB2
    end
```

> Este diagrama representa uma arquitetura híbrida onde a aplicação é acessada via API Gateway e distribuída entre EC2, containers e funções serverless, com comunicação com bancos na nuvem e no ambiente local (on-premise).
