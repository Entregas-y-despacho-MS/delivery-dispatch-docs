# 📦 Delivery Dispatch Docs — Microservicio de Entregas y Despachos

[![Documentación](https://img.shields.io/badge/Docs-Especificación%20Técnica-blue.svg)](./especificacion-sistema.md)
[![Requerimientos](https://img.shields.io/badge/RF-62%20Requerimientos-green.svg)](./requerimientos-funcionales.md)
[![Jira](https://img.shields.io/badge/Jira-Sprint%201--4-0052CC.svg)](./jira/PROJECT_BACKLOG.md)
[![ERP Corporativo](https://img.shields.io/badge/ERP-Grupo%20H-orange.svg)](#-contexto-y-alcance)

Repositorio central de **especificación formal, arquitectura de software, requerimientos funcionales y no funcionales, diagramas de interoperabilidad y gestión ágil** del microservicio de **Gestión de Entregas y Despachos (Última Milla)** para la cadena de retail/supermercados (Grupo H — Caso de estudio: *Hipermaxi / Dismac*).

---

## 👥 Equipo del Proyecto

| Nombre Completo | Rol |
|---|---|
| **Pardo Romano Alejandro Miguel** | Product Owner / Líder de Equipo |
| **Riveros Soria Joan Marcelo** | Scrum Master |
| **Huaycho Clavel Jaime Ignacio** | Dev Backend |
| **Mendoza Choque Sergio Alexander** | Dev Frontend |

*Universidad Católica Boliviana "San Pablo" — Regional La Paz, Bolivia*  
🔗 **Jira:** [Tablero de Proyecto Jira](https://joanmarceloriverossoria.atlassian.net/jira/software/projects/ES/boards/100/timeline)

---

## 📚 Estructura de la Documentación

```text
delivery-dispatch-docs/
├── especificacion-sistema.md        # Memoria técnica, arquitectura y modelo de dominio
├── requerimientos-funcionales.md    # Catálogo de los 62 RF (Usuario y Administrativos)
├── requerimientos-no-funcionales.md # Matriz RNF (SLA, offline-first, seguridad)
├── interoperabilidad-y-flujos.md    # Contratos REST con ERP y diagramas de secuencia
├── jira/                            # Planificación ágil y trazabilidad
│   ├── PROJECT_BACKLOG.md           # Backlog general, épicas y criterios GWT
│   ├── SPRINT_1.md                  # Sprint 1: Arquitectura base y autenticación
│   ├── SPRINT_2.md                  # Sprint 2: Despachos, ruteo y app móvil
│   ├── SPRINT_3.md                  # Sprint 3: Modo offline, tracking y evidencias
│   └── SPRINT_4.md                  # Sprint 4: Reportes, auditoría y cierre
└── README.md                        # Índice general (este archivo)