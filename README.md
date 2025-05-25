# 🛍️ Shopify AI Env

Un'applicazione Python per l'elaborazione automatica degli ordini Shopify tramite webhooks.

![Python](https://img.shields.io/badge/python-3.11-blue)
![Flask](https://img.shields.io/badge/flask-2.3-green)
![Docker](https://img.shields.io/badge/docker-ready-blue)
![Kubernetes](https://img.shields.io/badge/kubernetes-ready-blue)

## 🚀 Quick Start

### 1. Setup Locale
```bash
# Clona repository
git clone https://github.com/your-username/shopify-ai-env.git
cd shopify-ai-env

# Installa dipendenze
pip install -r requirements.txt

# Crea directory necessarie
mkdir -p logs storage/orders storage/temp

# Configura ambiente
cp .env.example .env
# Modifica .env con le tue credenziali Shopify
```

### 2. Avvio Sviluppo
```bash
python app/main.py
```

### 3. Setup Webhooks
```bash
curl -X POST http://localhost:5000/admin/setup
```

## 🐳 Docker (Raccomandato)

### Sviluppo Locale
```bash
# Build e avvio
docker build -t shopify-order-processor .
docker run -p 5000:8000 --env-file .env shopify-order-processor
```

### Stack Completo
```bash
# Con PostgreSQL + Redis + Nginx
docker-compose up --build
```

## ☸️ Kubernetes Deployment

### 1. Prepara Secrets
```bash
# Crea secret con le tue credenziali
kubectl create secret generic shopify-secret \
  --from-literal=SHOPIFY_API_KEY=your_key \
  --from-literal=SHOPIFY_API_SECRET=your_secret \
  --from-literal=SHOPIFY_ACCESS_TOKEN=your_token \
  --from-literal=SHOP_URL=your-shop.myshopify.com \
  --from-literal=WEBHOOK_BASE_URL=https://your-domain.com
```

### 2. Deploy
```bash
# Applica tutti i manifests
kubectl apply -f k8s/
```

### 3. Verifica
```bash
# Controlla pod
kubectl get pods -l app=shopify-order-processor

# Controlla servizi
kubectl get services

# Controlla ingress
kubectl get ingress
```

## ⚙️ Configurazione

### Credenziali Shopify

**App Privata (più semplice):**
1. Admin Shopify → Apps → "Develop apps"
2. Crea nuova app privata
3. Abilita Admin API access
4. Scopes: `read_orders`, `write_orders`

**App Pubblica:**
1. [Shopify Partners Dashboard](https://partners.shopify.com)
2. Crea nuova app
3. Ottieni API Key e Secret

### File .env
```bash
SHOPIFY_API_KEY=your_api_key
SHOPIFY_API_SECRET=your_api_secret
SHOPIFY_ACCESS_TOKEN=your_access_token
SHOP_URL=your-shop.myshopify.com
WEBHOOK_BASE_URL=https://your-domain.com
```

## 🎯 Personalizzazione

Modifica `app/services/order_processor.py` per aggiungere la tua logica:

```python
def _execute_business_logic(self, order, webhook_event):
    """🎯 INSERISCI LA TUA LOGICA QUI"""
    
    actions = []
    
    # Esempio: Ordini VIP
    if order.total_price > 100:
        actions.append(self._handle_vip_order(order))
    
    # Esempio: Prodotti speciali  
    special_products = self._check_special_products(order.items)
    if special_products:
        actions.append(self._handle_special_products(order, special_products))
    
    # 🎯 AGGIUNGI LA TUA LOGICA:
    # - Email personalizzate
    # - Aggiornamento CRM/ERP
    # - Notifiche Slack/Discord
    # - Integrazioni esterne
    
    return actions
```

## 📊 API Endpoints

| Endpoint | Descrizione |
|----------|-------------|
| `GET /` | Informazioni app |
| `GET /health` | Health check |
| `POST /webhook/orders_create` | Webhook nuovi ordini |
| `GET /admin/config` | Configurazione |
| `POST /admin/setup` | Setup webhooks |
| `GET /admin/orders/stats` | Statistiche |

## 🔄 Build e Deploy

### Build Docker Image
```bash
docker build -t shopify-order-processor:latest .
```

### Deploy su Kubernetes
```bash
# Crea namespace
kubectl create namespace shopify

# Deploy app
kubectl apply -f k8s/ -n shopify

# Verifica
kubectl get all -n shopify
```

### Deploy su Cloud

**Google Cloud Run:**
```bash
# Build e push
docker build -t gcr.io/your-project/shopify-order-processor .
docker push gcr.io/your-project/shopify-order-processor

# Deploy
gcloud run deploy shopify-order-processor \
  --image gcr.io/your-project/shopify-order-processor \
  --platform managed \
  --allow-unauthenticated
```

**AWS ECS/Fargate:**
```bash
# Build e push ad ECR
aws ecr get-login-password | docker login --username AWS --password-stdin your-account.dkr.ecr.region.amazonaws.com
docker build -t shopify-order-processor .
docker tag shopify-order-processor:latest your-account.dkr.ecr.region.amazonaws.com/shopify-order-processor:latest
docker push your-account.dkr.ecr.region.amazonaws.com/shopify-order-processor:latest
```

## 🧪 Testing

```bash
# Test dell'applicazione
pytest

# Test ordine fittizio
curl -X POST http://localhost:5000/admin/test/order

# Health check
curl http://localhost:5000/health/detailed
```

## 📁 Struttura Progetto

```
shopify-order-processor/
├── app/                        # Codice applicazione
│   ├── services/
│   │   └── order_processor.py  # 🎯 LOGICA PERSONALIZZABILE
│   ├── api/routes/             # Endpoint REST
│   └── main.py                 # Entry point
├── docker/                     # Configurazione Docker
│   ├── Dockerfile              # Build container
│   └── docker-compose.yml      # Stack completo
├── k8s/                        # Kubernetes manifests
│   ├── deployment.yaml         # Deployment
│   ├── service.yaml            # Service
│   └── ingress.yaml            # Ingress
├── .env                        # Configurazione
└── requirements.txt            # Dipendenze
```

## 🔒 Sicurezza

- ✅ Validazione HMAC dei webhooks
- ✅ HTTPS obbligatorio (in produzione)
- ✅ Rate limiting via Nginx/Ingress
- ✅ Container non-root
- ✅ Secrets Kubernetes

## 📝 Logs

I log sono in `logs/shopify_app_YYYYMMDD.log` o visualizzabili con:

```bash
# Docker
docker logs shopify-order-processor

# Kubernetes
kubectl logs -f deployment/shopify-order-processor
```

## 🆘 Troubleshooting

**Webhook non ricevuti:**
- Verifica URL pubblico accessibile
- Controlla HMAC secret
- Testa con `ngrok` per sviluppo locale

**Errore connessione Shopify:**
- Verifica credenziali in `.env`
- Controlla scopes dell'app
- Test: `curl http://localhost:5000/health/detailed`

## 📄 Licenza

MIT License - Vedi [LICENSE](LICENSE)

---

**🎯 Focus:** Deploy e build solo con **Docker + YAML**, niente script shell!