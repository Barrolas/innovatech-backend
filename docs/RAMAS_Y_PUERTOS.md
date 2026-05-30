# Ramas y puertos

| Rama | Uso | Backend host | Frontend host |
|------|-----|--------------|---------------|
| **main** | Desarrollo estándar EP2 / AWS | 8080, 8081, 3306 | 80 |
| **deploy** | Local con otros contenedores ocupando puertos | 9080, 9081, 3307 | 9082 |

Si en `main` tienes conflicto de puertos, copia `.env.example.alt-ports` a `.env` o cambia a rama `deploy`.
