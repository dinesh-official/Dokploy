
Installation Guide :

  sudo su
  apt install docker.io
  curl -sSL https://dokploy.com/install.sh | sh


App Architecture 
-----------------



                 ┌────────────────────┐
                 │    Frontend UI     │
                 │  (Next.js/React)   │
                 └─────────┬──────────┘
                           │ HTTP/API
                           ▼
                 ┌────────────────────┐
                 │   Dokploy Backend  │
                 │   API + Workers    │
                 └─────────┬──────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
 ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
 │ PostgreSQL   │   │    Redis     │   │ Docker API   │
 │ Metadata DB  │   │ Queue/Cache  │   │ Swarm Engine │
 └──────────────┘   └──────────────┘   └──────┬───────┘
                                               │
                                               ▼
                                     ┌─────────────────┐
                                     │ Docker Swarm    │
                                     │ Service Layer   │
                                     └────────┬────────┘
                                              │
                   ┌──────────────────────────┼──────────────────────────┐
                   ▼                          ▼                          ▼
            App Containers             Traefik Proxy              Build Containers



--------------------------------------------------------------------------------------------------------------------



                           ┌──────────────────┐
                           │    Developer     │
                           └────────┬─────────┘
                                    │
                              Git Push/Webhook
                                    │
                                    ▼
                     ┌──────────────────────────┐
                     │         Dokploy          │
                     │ Deployment Controller    │
                     └──────────┬───────────────┘
                                │
                    Docker API / Swarm API
                                │
                                ▼
                 ┌─────────────────────────────┐
                 │       Docker Swarm          │
                 │   Orchestration Manager     │
                 └──────────┬──────────────────┘
                            │
        ┌───────────────────┼────────────────────┐
        │                   │                    │
        ▼                   ▼                    ▼
 ┌──────────────┐   ┌──────────────┐    ┌──────────────┐
 │ Service Mgmt │   │ Overlay Net  │    │ Secret Mgmt  │
 │ Replica Ctrl │   │ Service DNS  │    │ Docker Secret│
 └──────┬───────┘   └──────┬───────┘    └──────┬───────┘
        │                  │                   │
        └──────────────────┼───────────────────┘
                           ▼
               ┌─────────────────────────┐
               │     Worker Nodes        │
               │   Container Runtime     │
               └──────────┬──────────────┘
                          │
       ┌──────────────────┼──────────────────┐
       ▼                  ▼                  ▼
Frontend Container  Backend Container  Database Container
       │                  │                  │
       └──────────────────┼──────────────────┘
                          ▼
                 ┌──────────────────┐
                 │     Traefik      │
                 │ Ingress/Proxy    │
                 └────────┬─────────┘
                          │
                    HTTP / HTTPS
                          │
                          ▼
                        Users







