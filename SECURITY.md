# Security Policy

## Naturaleza del repo

Este repositorio contiene **únicamente documentación de gobernanza** (SPEC, ADRs, HANDOFFs, plantillas). No hay código ejecutable, dependencias npm, ni artefactos binarios. La superficie de ataque es la integridad documental.

## Modelo de amenaza considerado

- **Suplantación de identidad de owner** — alguien abre un PR malicioso modificando una decisión canónica (`D-NNN`) o un invariante de seguridad transversal. **Mitigación**: branch protection + `CODEOWNERS` + revisión humana obligatoria + squash merge auditado.
- **Inyección de referencias falsas** — un PR introduce URLs apuntando a recursos no oficiales (typosquatting de `atlax360.ai`). **Mitigación**: CI de link-check + test funcional `naming-convention` en repos consumidores.
- **Fuga de datos sensibles vía ejemplos** — un ejemplo de configuración incluye credenciales reales. **Mitigación**: regla CLAUDE.md global "nunca commit de secrets" + revisión en PR.

## Cómo reportar una vulnerabilidad o anomalía

Email: **jgcalvo@atlax360.com**

Incluir:

- Descripción del problema
- Fichero/sección afectada
- Severidad estimada (low / medium / high / critical)
- Sugerencia de fix si aplica

**No abrir issue público** para vulnerabilidades activas — esperar respuesta privada primero.

## Tiempo de respuesta esperado

- **Acuse de recibo**: 48h
- **Análisis inicial**: 7 días
- **Fix o decisión de no-fix justificada**: 30 días

Este SLA es aspiracional: el repo es mantenido por un único owner activo y el ritmo de respuesta depende de carga semanal.

## Disclosure responsable

Tras fix mergeado, el reporter puede ser acreditado en el CHANGELOG si así lo desea. Avisar en el reporte si se prefiere disclosure anónima.
