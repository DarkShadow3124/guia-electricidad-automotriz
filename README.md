# Manual de Electricidad Automotriz — sitio web

Página estática de descarga del manual (PDF) del curso de electricidad automotriz.

- **Dominio deseado:** guia.capygamerstore.com
- **Contenido:** `index.html` (una sola pantalla) + `manual-electricidad-automotriz.pdf`
- **Aviso:** la guía fue elaborada con ayuda de IA y no revisada por un especialista.

## Para montar en el servidor
Ver **DESPLIEGUE-ORACLE.md** — instrucciones paso a paso para Oracle Cloud (Ubuntu + Nginx + dominio + HTTPS).

## Estructura
```
index.html                          página principal
manual-electricidad-automotriz.pdf  el manual (~9 MB)
DESPLIEGUE-ORACLE.md                guía de despliegue
README.md                           este archivo
```

Es un sitio 100 % estático: no necesita backend, base de datos ni proceso de build. Basta servir estos archivos.
