# OpenSpec GitHub Workflows

Este directorio contiene automatizaciones para el proyecto OpenSpec.

## Workflows Disponibles

### openspec-validate.yml
Valida la estructura de cambios OpenSpec en cada PR.
- Verifica que exista openspec/config.yaml
- Valida la estructura de directorios

### openspec-generate.yml
Genera documentación automáticamente cuando se actualiza openspec/changes en main.
- Lista archivos de cambios
- Puede ejecutar herramientas de generación
- Hace commit de archivos generados (si los hay)

## Cómo Agregar Más Workflows

1. Crea un nuevo archivo `.yml` en este directorio
2. Define los eventos (`on`) que lo disparan
3. Configura los jobs y steps

## Secretos Necesarios

Si necesitas integración con APIs externas:
- Ve a Settings → Secrets and variables → Actions
- Agrega los secretos necesarios
- Refiere a ellos como `${{ secrets.TU_SECRETO }}`
