# Siguientes pasos EP2 — Instrucciones

Ya completaste **Fase A** (repos), **Fase B** (backend Docker) y **Fase C** (frontend Docker).  
Sigue en este orden.

---

## FASE C — Frontend ✅ (hecho)

**Qué se hizo:**
- `src/config/api.js` — URLs relativas en Docker, absolutas en `npm run dev`
- Dockerfile multi-stage (Node → Nginx)
- `nginx.conf.template` — proxy `/api/v1/ventas` y `/api/v1/despachos` → backend
- `docker-compose.yml` — front en **http://localhost:9082**

**Probar (backend debe estar arriba en 9080/9081):**

```powershell
# Terminal 1 — backend
cd "e:\DOWNLOADS\Proyecto Semestral - DevOps - EV2"
docker compose up -d

# Terminal 2 — frontend
cd "e:\DOWNLOADS\Proyecto Semestral - DevOps - EV2\front_despacho"
copy .env.example .env
docker compose up --build -d
```

Abrir **http://localhost:9082** → F12 → Network → ver requests a `/api/v1/ventas` con status 200.

**Capturas sugeridas:** app en 9082, DevTools Network, Dockerfile + nginx template.

---

## FASE D — AWS Learner Lab (Consola web)

Haz todo con **Learner Lab → Start Lab → AWS** (luz verde).

### D.1 Security Groups

**SG Front (`sg-front`):**

| Tipo | Puerto | Origen |
|------|--------|--------|
| HTTP | 80 | 0.0.0.0/0 |
| SSH | 22 | My IP |

**SG Back (`sg-back`):**

| Tipo | Puerto | Origen |
|------|--------|--------|
| TCP | 8080 | sg-front |
| TCP | 8081 | sg-front |

Consola: **VPC** → **Security groups** → **Create security group**.

### D.2 EC2 Frontend

1. **EC2** → **Launch instance**
2. Amazon Linux 2023, `t2.micro`
3. Subred **pública**, auto-assign public IP **Enable**
4. SG: `sg-front`
5. Key pair: crear y descargar `.pem`
6. **Advanced details** → **IAM instance profile**: rol con `AmazonEC2ContainerRegistryReadOnly`
7. Launch → **Elastic IP** → asociar → anotar **IP pública**

### D.3 EC2 Backend

1. Misma AMI, subred **privada**, sin IP pública
2. SG: `sg-back`
3. Mismo IAM profile (ECR read)
4. Anotar **IP privada** (ej. 10.0.x.x)

### D.4 ECR — 3 repositorios

Consola → buscar **ECR** → **Create repository**:

- `innovatech-frontend`
- `innovatech-ventas`
- `innovatech-despachos`

### D.5 Docker en cada EC2 (una vez)

**EC2** → instancia Front → **Connect** → **EC2 Instance Connect**:

```bash
sudo yum update -y
sudo yum install -y docker
sudo systemctl start docker && sudo systemctl enable docker
sudo usermod -aG docker ec2-user
sudo mkdir -p /usr/local/lib/docker/cli-plugins
sudo curl -SL "https://github.com/docker/compose/releases/latest/download/docker-compose-linux-$(uname -m)" \
  -o /usr/local/lib/docker/cli-plugins/docker-compose
sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
sudo mkdir -p /home/ec2-user/app
```

Cerrar sesión, volver a entrar, probar: `docker --version`

Repetir en EC2 Back (Session Manager o desde consola si tiene acceso).

### D.6 Antes de cada prueba de pipeline

Learner Lab → **AWS Details** → copiar Access Key, Secret, **Session Token** → actualizar GitHub Secrets (Fase E).

---

## FASE E — GitHub Actions (CI/CD)

### E.1 Secrets — repo [innovatech-backend](https://github.com/Barrolas/innovatech-backend)

**Settings** → **Secrets and variables** → **Actions** → **New repository secret**

| Secret | Valor |
|--------|-------|
| `AWS_ACCESS_KEY_ID` | Learner Lab |
| `AWS_SECRET_ACCESS_KEY` | Learner Lab |
| `AWS_SESSION_TOKEN` | Learner Lab |
| `AWS_REGION` | ej. `us-east-1` |
| `ECR_REPOSITORY_VENTAS` | `innovatech-ventas` |
| `ECR_REPOSITORY_DESPACHOS` | `innovatech-despachos` |
| `EC2_HOST` | IP privada EC2 Back (o pública si SSH directo) |
| `EC2_SSH_KEY` | Contenido completo del `.pem` |
| `EC2_USER` | `ec2-user` |
| `DB_NAME` | `innovatech` |
| `DB_USERNAME` | `innovatech` |
| `DB_PASSWORD` | tu contraseña |

### E.2 Secrets — repo [innovatech-frontend](https://github.com/Barrolas/innovatech-frontend)

| Secret | Valor |
|--------|-------|
| (AWS iguales) | |
| `ECR_REPOSITORY` | `innovatech-frontend` |
| `EC2_HOST` | IP **pública** EC2 Front |
| `VITE_API_VENTAS_URL` | vacío |
| `VITE_API_DESPACHOS_URL` | vacío |
| `VENTAS_UPSTREAM` | `http://IP_PRIVADA_BACK:8080` |
| `DESPACHOS_UPSTREAM` | `http://IP_PRIVADA_BACK:8081` |

### E.3 Crear workflow backend

Archivo: `.github/workflows/deploy.yml` en raíz repo backend.

Trigger: push a rama `deploy`.

Pasos: checkout → AWS creds → ECR login → build/push ventas → build/push despachos → SSH EC2 → `docker compose up -d`

**Commit:** `[ CI ]: Agregar workflow deploy backend en rama deploy`

### E.4 Crear workflow frontend

Archivo: `front_despacho/.github/workflows/deploy.yml`

Build con `VITE_*` vacíos → push ECR → SSH EC2 Front → deploy contenedor nginx.

**Commit:** `[ CI ]: Agregar workflow deploy frontend en rama deploy`

### E.5 Probar pipeline

1. Renovar Secrets AWS (lab activo)
2. `git push origin deploy` en backend
3. GitHub → **Actions** → esperar verde
4. Igual en frontend
5. Captura: workflow verde + imágenes en ECR

---

## FASE F — Verificación en AWS

### F.1 Backend EC2

```bash
docker ps
curl http://localhost:8080/api/v1/ventas
curl http://localhost:8081/api/v1/despachos
```

### F.2 Frontend EC2

Navegador: `http://IP_PUBLICA_FRONT`  
DevTools → Network → `/api/v1/ventas` → 200

### F.3 Integración (IE7)

- Solo Front accesible desde internet
- Back solo desde SG Front
- App carga datos reales desde APIs

### F.4 Demo presentación

1. Cambio mínimo en código
2. `git push origin deploy`
3. Mostrar Actions ejecutándose
4. Refrescar navegador con IP pública

---

## FASE G — README + AVA

En cada repo, README con:

1. Cómo clonar y `docker compose up`
2. Diagrama arquitectura (Mermaid)
3. Justificación named volume
4. Explicación nginx proxy (Front → Back privado)
5. Lista de GitHub Secrets (nombres)
6. Links a capturas o evidencias

Entregar en **AVA** links a ambos repos + PowerPoint presentación.

---

## Puertos resumen (local)

| Servicio | URL local |
|----------|-----------|
| Ventas | http://localhost:9080 |
| Despachos | http://localhost:9081 |
| MySQL | localhost:3307 |
| Frontend | http://localhost:9082 |

## Commits pendientes típicos

```
[ CI ]: Agregar workflow deploy backend en rama deploy
[ CI ]: Agregar workflow deploy frontend en rama deploy
[ DOCS ]: Completar README con arquitectura y CI/CD
```

---

*Tu siguiente acción manual: **Fase D** en Learner Lab (EC2 + ECR + SG). Cuando tengas IPs y ECR, avisa y armamos los workflows (Fase E).*
