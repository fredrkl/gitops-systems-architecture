# GitOps Systems Architecture

This is the GitOps architecture I created at If-insurance, a rather large insurance company in the Nordics. It is based on the [GitOps principles](https://www.weave.works/technologies/gitops/) and the [GitOps FAQ](https://www.weave.works/blog/gitops-faq). Please read the blog post and explanation of the setup on [my blog](https://fredrkl.com/blog/infrastructure-as-code-vs-gitops-a-real-world-example/).

The diagram is created with [mermaid.js](https://mermaid.js.org/).

```mermaid
flowchart LR
    %% The entities
    bko(platform IaC)
    ssc(Sealed Secret Controller)
    flux(Flux Controller)

    %% The Git repositories CRDs
    gr("GitRepository</br>#60;K8s CRD#62;")
    grsystems("GitRepository</br>#60;K8s CRD#62;")
    grcardissuer("GitRepository</br>#60;K8s CRD#62;")
    grfrauddetection("GitRepository</br>#60;K8s CRD#62;")

    %% The Kustomization CRDs
    kustomize("Kustomization</br>#60;K8s CRD#62;")
    kustomizesystemscardissuer("Kustomization</br>#60;K8s CRD#62;")
    kustomizesystemsfrauddetection("Kustomization</br>#60;K8s CRD#62;")
    
    prometheus("Prometheus")
    grafana("Grafana")
    linkerd("Linkerd")
    alertManager("Alert Manager")
    kustomizesystems("Kustomization</br>#60;K8s CRD#62;")

    %% Repos
    platformsystemsdb[(Platform Systems\nManifest Repo)]
    systemsdb[(Payment Systems\nManifest Repo)]
    cardIssuerdb[(CardIssuer\nManifest Repo)]
    fraudDetectiondb[(Fraud detection\nManifest Repo)]

    %% Repo responsible
    platformteam(("#128104;</br>Platform Team"))
    cardissuerteam(("#128104;</br>Card Issuer Team"))
    frauddetestionteam(("#128104;</br>Fraud Detection Team"))

    platformteam-. responsibe for .->platformsystemsdb
    platformteam-. responsibe for .->systemsdb

    cardissuerteam-. responsibe for .-> cardIssuerdb
    frauddetestionteam-. responsibe for .->fraudDetectiondb

    %% The flow
    subgraph IaC Kickoff
        bko--"#9312; Install"-->ssc
        bko--"#9313; Install"-->flux
        bko--"#9314; Initializing"-->gr
        bko--"#9314; Initializing"-->kustomize
        kustomize--"Uses"-->gr
        subgraph Instances refleting the environment
            kustomize
            gr
        end
    end

    gr--"Pulls inn from GitRepo"-->platformsystemsdb
    PlatformSystems-.->platformsystemsdb
    
    subgraph PlatformSystems
        grafana
        prometheus
        alertManager
        linkerd

        kustomizesystems--"Uses"-->grsystems
        subgraph SystemSync
            kustomizesystems
            grsystems
        end
    end

    grsystems--"Pulls in from GitRepo"-->systemsdb
    Systems-.->systemsdb

    subgraph Systems
        pv("Persistent Volumes")
        cr("Cluster Roles")
        np("Networking Policies")
        kustomizesystemscardissuer--"Uses"-->grcardissuer
        
        subgraph CarsIssuer-System
            kustomizesystemscardissuer
            grcardissuer
        end

        kustomizesystemsfrauddetection--"Uses"-->grfrauddetection
        subgraph Fraud detection-System
            kustomizesystemsfrauddetection
            grfrauddetection
        end
    end

    grcardissuer--"Pulls in from GirRepo"-->cardIssuerdb
    grfrauddetection--"Pulls in from GitRepo"-->fraudDetectiondb

```

We use [kustomize](https://kustomize.io/) extensively. Please read on how we use it together with branches to control rolling out changes to different environments [here](./kustomize.md). The Kustomization CRD boxes are the configuration of how the Git changes are applied. Please see [Kustomization CRD](https://fluxcd.io/flux/components/kustomize/kustomization/) for more information and examples.
