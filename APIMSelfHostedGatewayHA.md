# Deploy Azure API Management Self-Hosted Gateway in High Availability using Helm

## Prerequisites

- Existing Kubernetes cluster (multi-node recommended)
- Azure API Management instance (Developer or Premium tier)
- Provisioned self-hosted gateway resource in API Management
- Helm v3 installed

## Steps

### 1. Add the Helm Repository

```sh
helm repo add azure-apim-gateway https://azure.github.io/api-management-self-hosted-gateway/helm-charts/
helm repo update
```

### 2. Prepare Gateway Configuration

- In the Azure portal, go to your API Management instance.
- Navigate to **Gateways** and select your self-hosted gateway resource.
- Under **Deployment**, generate a new token and note the **Token** and **Configuration URL**.

### 3. Deploy the Gateway with High Availability

```sh
helm install azure-api-management-gateway \
	--set gateway.configuration.uri='<your configuration url>' \
	--set gateway.auth.key='<your token>' \
	--set highAvailability.enabled=true \
	azure-apim-gateway/azure-api-management-gateway
```

- To expose the gateway externally, add: `--set service.type=LoadBalancer`

### 4. Verify Deployment

```sh
kubectl get pods
kubectl get services
kubectl logs <gateway-pod-name>
```

### 5. High Availability Best Practices

- Use clusters with availability zones for node-level HA.
- Apply Pod Topology Spread Constraints or Pod Anti-Affinity for zone distribution.
- Set up Pod Disruption Budgets to maintain minimum pod availability.

## References

- [Deploy self-hosted gateway to Kubernetes with Helm](https://learn.microsoft.com/en-us/azure/api-management/how-to-deploy-self-hosted-gateway-kubernetes-helm)
- [High availability guidance](https://learn.microsoft.com/en-us/azure/api-management/how-to-self-hosted-gateway-on-kubernetes-in-production#high-availability)
- [Provision a self-hosted gateway in Azure API Management](https://learn.microsoft.com/en-us/azure/api-management/api-management-howto-provision-self-hosted-gateway)
- [API Management gateways overview](https://learn.microsoft.com/en-us/azure/api-management/api-management-gateways-overview#managed-and-self-hosted)
