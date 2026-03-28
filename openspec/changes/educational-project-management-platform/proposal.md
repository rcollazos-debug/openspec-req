# Proposal: educational-project-management-platform

## Intent

Los talleres educativos de construcción de ciudades con legos enfrentan un problema crítico de gestión y seguimiento en tiempo real. Actualmente, los instructores deben manejar manualmente el progreso de 30 estudiantes distribuidos en 5 zonas de construcción, sin visibilidad clara del cumplimiento de objetivos ni capacidad de reacción ante desviaciones del plan. Esta falta de herramientas digitalizadas genera pérdida de tiempo valioso (cada 10 minutos representa 1 mes del proyecto), descoordinación entre equipos, y dificultad para evaluar el aprendizaje de gestión de proyectos que es el objetivo pedagógico central.

La ausencia de un sistema integrado impide la simulación realista de escenarios empresariales, limita la capacidad de generar reportes de desempeño estudiantil, y reduce significativamente el impacto educativo del taller. Los estudiantes pierden la oportunidad de experimentar herramientas profesionales de gestión de proyectos en un entorno controlado y gamificado.

Esta plataforma transformará el taller en una experiencia inmersiva y profesional, proporcionando valor educativo medible y preparando a los estudiantes con competencias reales de gestión de proyectos aplicables en su vida profesional.

## Scope In

- Sistema de autenticación con correo/contraseña y enlaces de bienvenida automatizados vía Gmail API
- Gestión completa de roles y permisos (Administrador, Director, Coordinadores, Integrantes) con funcionalidades específicas por rol
- Módulo de planificación por zonas con creación de meses, hitos y entregables por Coordinadores
- Cálculo automático de porcentaje de cumplimiento por zona (20% cada zona del total del proyecto)
- Sistema completo de gestión de tareas con flujo de estados (Creada → Asignada → En curso → En revisión → Finalizada)
- Pantalla de proyección en tiempo real con cuenta regresiva, mes actual, progreso por zona e impresión automática cada 5 minutos
- Motor de riesgos aleatorios parametrizables con visualización en pantalla de proyección
- Generación y gestión de actas de inicio por Coordinador con consolidación automática y flujo de aprobación secuencial
- Integración completa con Google Drive API para gestión de archivos por zona
- Integración con Gmail API para notificaciones automáticas y recordatorios
- Dashboard administrativo con métricas de cumplimiento, historial y reportes descargables
- Vistas Gantt por zona (Coordinadores) y consolidada (Director) con visualización de hitos y entregables
- Sistema de auditoría completo con historial de cambios (quién, qué, cuándo) en seguimiento de entregables
- Notificaciones automáticas por correo y en aplicación con alertas de tareas próximas a vencer
- Base de datos PostgreSQL con esquemas optimizados para los 11 dominios identificados

## Scope Out

- Integración con otras plataformas de almacenamiento que no sean Google Drive (Dropbox, OneDrive, etc.)
- Funcionalidades de video conferencia o comunicación en tiempo real entre participantes
- Sistema de gamificación con puntos, badges o rankings competitivos entre estudiantes
- Módulo de evaluación académica o calificaciones numéricas de desempeño estudiantil
- Funcionalidades de gestión financiera o presupuestaria del proyecto simulado
- Integración con sistemas LMS existentes de la Universidad Autónoma de Occidente
- Soporte para múltiples talleres simultáneos o gestión de diferentes cohortes
- Aplicación móvil nativa (solo versión web responsive)
- Funcionalidades de inteligencia artificial o machine learning para predicción de riesgos
- Sistema de backup automático o recuperación ante desastres
- Soporte multiidioma (solo español)

## Approach

Se implementará una arquitectura web moderna basada en el patrón MVC con separación clara de responsabilidades por dominios OpenSpec. La aplicación utilizará PostgreSQL como base de datos principal con esquemas normalizados para cada dominio (auth, users, roles, notifications, projects, tasks, risks, files, reports, audit, api).

El frontend será una aplicación web responsive con actualización en tiempo real mediante WebSockets para la pantalla de proyección. Se implementará un sistema de roles basado en permisos granulares que permita la escalabilidad de funcionalidades por tipo de usuario.

La integración con Google APIs se realizará mediante OAuth 2.0 para autenticación segura y manejo de tokens de acceso. El motor de riesgos utilizará algoritmos de probabilidad parametrizables para simular escenarios realistas de gestión de proyectos.

Se adoptará un enfoque de desarrollo iterativo con entregables funcionales cada sprint, priorizando primero la funcionalidad core (autenticación, roles, tareas) y posteriormente las integraciones externas y funcionalidades avanzadas.

## Success Criteria

- [ ] El sistema autentica exitosamente a 30 usuarios simultáneos con roles diferenciados en menos de 3 segundos por login
- [ ] Los Coordinadores pueden crear y gestionar hitos/entregables con cálculo automático de cumplimiento por zona en tiempo real
- [ ] La pantalla de proyección actualiza información cada 10 segundos y genera impresiones automáticas cada 5 minutos al finalizar cada mes simulado
- [ ] El motor de riesgos ejecuta eventos aleatorios según parámetros configurados con visualización inmediata en pantalla de proyección
- [ ] Las integraciones con Google Drive y Gmail funcionan sin interrupciones durante sesiones de 3 horas continuas
- [ ] Los reportes de cumplimiento se generan y descargan en formato PDF en menos de 10 segundos
- [ ] El sistema mantiene un historial completo de auditoría con capacidad de rastrear todos los cambios realizados por cada usuario
- [ ] Las notificaciones automáticas se envían con 100% de confiabilidad tanto por correo como en la aplicación
- [ ] Los dashboards Gantt muestran información actualizada en tiempo real para Coordinadores y Director
- [ ] El sistema soporta 30 usuarios concurrentes sin degradación de performance durante las 3 semanas del taller

## Risks & Mitigations

| Riesgo | Impacto | Mitigación |
|--------|---------|------------|
| Límites de API de Google Drive/Gmail durante uso intensivo | Alto | Implementar cache local, rate limiting y cuentas de servicio múltiples con rotación automática |
| Pérdida de conectividad a internet durante el taller afectando integraciones críticas | Alto | Desarrollar modo offline para funcionalidades core con sincronización automática al restaurar conexión |
| Sobrecarga de base de datos con 30 usuarios concurrentes y actualizaciones en tiempo real | Medio | Optimización de queries, indexación estratégica y implementación de connection pooling |
| Complejidad del cálculo de porcentajes de cumplimiento por zona generando inconsistencias | Medio | Implementar validaciones cruzadas, logs detallados de cálculos y funcionalidad de recálculo manual |
| Fallos en la pantalla de proyección interrumpiendo la dinámica del taller | Alto | Sistema de respaldo con proyección desde múltiples dispositivos y modo de recuperación rápida |