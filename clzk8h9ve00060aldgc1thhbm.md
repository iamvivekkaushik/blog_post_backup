---
title: "Step-by-Step Guide: Nginx Ingress Setup on AKS"
datePublished: Wed Aug 07 2024 19:19:54 GMT+0000 (Coordinated Universal Time)
cuid: clzk8h9ve00060aldgc1thhbm
slug: step-by-step-guide-nginx-ingress-setup-on-aks
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1723011750409/aa9a8b7a-d25e-4843-9e1a-68b00990d30b.png

---

Recently I was tasked with setting up kubernetes cluster with an Nginx Ingress. As a newcomer to Kubernetes, I tackled this setup and learned a thing or two along the way. Let me walk you through my experience and share some helpful tips.

**The Goal:** Set up a Kubernetes Cluster with Nginx Ingress

Here's where things got interesting. After setting up my cluster, I hit a snag: I couldn't get a public IP assigned to my Nginx ingress. All I got was a private IP, which wasn't very useful for my needs.

After some extensive searching (and maybe a few sighs of frustration), I finally found the solution in the Nginx documentation ([https://kubernetes.github.io/ingress-nginx/deploy/](https://kubernetes.github.io/ingress-nginx/deploy/)). What a relief!

Before We Start...

Let's make sure we're on the same page. This guide assumes you've already configured your K8s cluster on kubectl. If you haven't, no problem! Just use `az login` to access your Azure account and use the connect command from your Azure AKS cluster. It's pretty straightforward.

### The Key Command

Here's the command that'll set up your Nginx ingress controller:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/cloud/deploy.yaml
```

Found this gem after going throught the documentation (maybe we should read it sometime) [https://kubernetes.github.io/ingress-nginx/deploy/#azure](https://kubernetes.github.io/ingress-nginx/deploy/#azure).

### Create a certificate Secret

Now, let's tackle the next exciting step in our K8s journey: creating a certificate Secret! 🔐.

You'll need to whip up a certificate for your domain. You've got two options here:

1. Snag a free certificate from Let's Encrypt
    
2. Purchase one from a certificate authority (fancy, I know!)
    

Once you've got your hands on that shiny certificate and key, Run this command:

```bash
kubectl create secret tls tls-conf --cert /path/to/fullchain.pem --key /path/to/privkey.pem --namespace your-namespace
```

Don't forget to swap out the `path` and `namespace` with your actual values. This command creates a secret object of type `kubernetes.io/tls`.

### Time to create a Ingress

Next up, we're creating an Ingress for your service. Think of it as the bouncer that decides who gets into your Kubernetes club.

Create a file called `ingress.yaml` and fill it with this yaml goodness:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-name
  namespace: your-namespace
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  tls:
    - hosts:
        - your-domain.something
      secretName: tls-conf
  rules:
    - host: your-domain.something
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: your-service
                port:
                  number: your-service-port
  ingressClassName: nginx
```

Now, before you apply this yaml, make sure to replace `your-namespace`, `your-domain.something`, `your-service`, and `your-service-port` with your actual values.

Once you've customized your yaml, apply it using kubectl. Voila! You've just set up an Ingress for your service using the following command.

```bash
kubectl apply -f /path/to/ingress.yaml
```

Replace the `/path/to/ingress.yaml` with wherever you saved your ingress.yaml.

And that's it you should have your nginx ingress up and running with a public IP assigned to it.

### Bonus Round: Customizing Your Nginx Config 🛠️

What if you want to tweak some Nginx settings? Maybe you're looking to adjust the `client_max_body_size` or other fancy parameters? Well, you're in luck!

1. Head over to the `config` objects in your Kubernetes dashboard.
    
2. Filter by the `ingress-nginx` namespace.
    
3. find and edit th `ingress-nginx-controller`
    
4. Once you're happy with your edits, apply them. A complete list of available configuration options can be found at: [https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/).