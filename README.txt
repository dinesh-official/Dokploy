
Installation Guide :

  sudo su
  apt install docker.io
  curl -sSL https://dokploy.com/install.sh | sh



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



------------------------------------------------------------------------------------------------------------------------


                    Developer
                        │
                 git push origin main
                        │
                        ▼
             ┌────────────────────┐
             │  GitHub / GitLab   │
             └─────────┬──────────┘
                       │ Webhook
                       ▼
             ┌────────────────────┐
             │      Dokploy       │
             │ Deployment Engine  │
             └─────────┬──────────┘
                       │
          ┌────────────┼────────────┐
          │            │            │
          ▼            ▼            ▼
    Clone Repo     Read Config    Queue Build
          │            │            │
          └────────────┼────────────┘
                       ▼
             ┌────────────────────┐
             │   Build Container  │
             │  docker build      │
             └─────────┬──────────┘
                       │
                 Create Image
                       │
                       ▼
             ┌────────────────────┐
             │ Docker Swarm       │
             │ Service Deployment │
             └─────────┬──────────┘
                       │
       ┌───────────────┼────────────────┐
       ▼               ▼                ▼
Frontend Service  Backend Service  Database Service
       │               │                │
       └───────────────┼────────────────┘
                       ▼
               Docker Overlay Network
                       │
                       ▼
             ┌────────────────────┐
             │      Traefik       │
             │ Reverse Proxy SSL  │
             └─────────┬──────────┘
                       │
                 HTTP / HTTPS
                       │
                       ▼
                    Users


