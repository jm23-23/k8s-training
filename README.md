# Kubernetes Training

Zbiór manifestów Kubernetes do nauki kluczowych mechanizmów klastra — schedulingu, sieci, zarządzania zasobami, sekretów oraz GitOps.

## Struktura repozytorium

### `modul1-scheduling/` — Sterowanie rozmieszczeniem podów (Taints & Tolerations)
- **`gpu-pod.yaml`** — Pod (`ml-pod`) z `toleration` na taint `sku=gpu:NoSchedule`, dzięki czemu może trafić na dedykowany (np. GPU) węzeł.
- **`isolated-pod.yaml`** — Zwykły pod (`zwykly-pod`) bez tolerancji, służy jako punkt odniesienia (nie wejdzie na "skażony" węzeł).

### `modul1-antiaffinity/` — Anti-Affinity (rozpraszanie podów)
- **`frontend-anti-affinity.yaml`** — Deployment `frontend-ha` (2 repliki) z **twardą** regułą `requiredDuringScheduling` — pody nie mogą współdzielić tej samej strefy (`topology.kubernetes.io/zone`).
- **`frontend-preferred-affinity.yaml`** — Deployment `frontend-soft` (3 repliki) z **miękką** regułą `preferredDuringScheduling` (waga 100) — rozpraszanie preferowane, ale niewymuszane.

### `modul2-networking/` — Izolacja sieciowa (Network Policies)
- **`01-infrastructure.yaml`** — Infrastruktura testowa: namespace'y `frontend-ns` i `backend-ns` oraz pody `frontend-app`, `backend-app` i `database-pod` (Redis).
- **`02-network-policy.yaml`** — `NetworkPolicy` `db-allow-only-backend` — do bazy (Redis, port 6379) ma dostęp wyłącznie pod `backend` z `backend-ns`.

### `modul3-resources/` — Zasoby i autoskalowanie
- **`app-to-scale.yaml`** — Deployment `scalable-app` z requests/limits CPU i pamięci oraz `HorizontalPodAutoscaler` (1–5 replik, skalowanie przy >50% CPU).
- **`oom-test.yaml`** — Pod `oom-target-pod` (`stress-ng`) z limitem pamięci 20Mi przy próbie alokacji większej ilości — demonstracja OOMKill oraz QoS `Guaranteed`.

### `modul4-secrets/` — External Secrets Operator
- **`01-secret-store.yaml`** — `SecretStore` `fake-vault` (provider `fake`) symulujący zewnętrzny magazyn poświadczeń.
- **`02-external-secret.yaml`** — `ExternalSecret` synchronizujący dane z `fake-vault` do natywnego Secreta K8s `local-k8s-secret` (`DB_USER`, `DB_PASSWORD`).

### Pliki w katalogu głównym
- **`argocd-application.yaml`** — Definicja `Application` dla ArgoCD (GitOps). Synchronizuje katalog `modul3-resources` z repozytorium do klastra z włączonym `prune` i `selfHeal`.

## Wymagania

Niektóre moduły wymagają dodatkowych komponentów w klastrze:
- **modul3** — zainstalowany Metrics Server (dla HPA).
- **modul4** — zainstalowany External Secrets Operator.
- **argocd-application.yaml** — działający ArgoCD.

## Użycie

```bash
# Zastosowanie pojedynczego modułu
kubectl apply -f modul2-networking/

# Lub konkretnego manifestu
kubectl apply -f modul1-scheduling/gpu-pod.yaml
```
