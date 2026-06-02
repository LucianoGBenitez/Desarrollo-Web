# Portal Financiero EEH

## Middleware de Conciliación Contable, Automatización de Transferencias y Sincronización de Datos Maestros

**Trabajo Práctico - Desarrollo Web**
**Carrera:** Analista Funcional en Sistemas Informáticos
**Empresa:** Essential Energy Holding (EEH)

---
## Tecnologías Utilizadas

- PHP 8.3
- MySQL
- SAP Business One Service Layer
- Interbanking API
- Google OAuth 2.0

## Arquitectura General

Interbanking API
        ↓
Portal Financiero EEH
        ↓
SAP Business One
        ↓
Base de Datos MySQL
---
---

## Tabla de Contenidos

* [Descripción del Sistema](#descripción-del-sistema)
* [Problemática](#problemática-en-el-contexto-real)
* [Stakeholders](#identificación-de-stakeholders)
* [Requisitos Funcionales](#requisitos-funcionales-rf)
* [Requisitos No Funcionales](#requisitos-no-funcionales-rnf)
* [Historias de Usuario](#historias-de-usuario)
* [Caso de Uso Principal](#caso-de-uso-principal)
* [Modelo de Datos](#modelo-entidad-relación)
* [Referencias](#referencias-y-fuentes-bibliográficas)

---

# Descripción del Sistema

El **Portal Financiero EEH** es una plataforma web corporativa desarrollada para centralizar, auditar y automatizar las operaciones del departamento de tesorería y contabilidad de **Essential Energy Holding (EEH)**.

El holding está compuesto por tres sociedades operativas:

* Rosario Bio Energy (RBE)
* Establecimiento El Albardón (EEA)
* Bioenergías Agropecuarias (BEA)

Cada una posee balances y entornos contables independientes.

---

# Problemática en el Contexto Real

Las operaciones financieras tradicionales presentan una desconexión entre los sistemas bancarios y los sistemas ERP corporativos. Las tareas rutinarias, como la conciliación de extractos bancarios diarios y el alta de cuentas para
transferencias masivas de fondos, suelen realizarse de manera manual.

Esto genera:

* Errores de transcripción manual en la carga de CBU.
* Inconsistencias contables por demoras en conciliaciones.
* Riesgo operativo en transferencias masivas.
* Procesos manuales repetitivos y propensos a errores.


---

# Identificación de Stakeholders

## Actores Internos

| Actor                 | Responsabilidad                      |
| --------------------- | ------------------------------------ |
| Operador de Tesorería | Cargar mensualmente las planillas de pago, previsualizar los lotes de datos y descargar el archivo TEF de ancho fijo de forma ágil y segura. |
| Analista Contable     | Conciliación de extractos bancarios, validando los asientos correctamente reflejados en la cuenta mayor de la BD de SAP correspondiente de forma inmediata  |
| Gerente de Finanzas   | Supervisión y control financiero: reduce los costos operativos, mitiga riesgos de auditoría contable y supevisa transferencias masivas mediante notificaciones rápidas y seguras.    |
| Administrador IT      | Infraestructura y monitoreo del correcto funcionamiento del servidor web, manitiene conexiones del Service Layer de SAP, resguarda las credenciales en el archivo de entorno y audita las ejecuciones asincronicas del sistema.          |

## Actores Externos

### Interbanking API Gateway

Proveedor de servicios bancarios para:

* Consulta de movimientos
* Transferencias electrónicas
* Validación financiera

### SAP Business One Service Layer

ERP encargado de:

* Registrar movimientos contables
* Administrar socios de negocio
* Gestionar cuentas bancarias

---

# Requisitos Funcionales (RF)

### RF-01

**Autenticación mediante Google OAuth y lista blanca corporativa.**
El ingreso al portal debe exigir autenticación con cuentas de Google pertenecientes obligatoriamente al dominio corporativo @albardonbio.com. Adicionalmente, el sistema validará el acceso contrastando el correo con una lista blanca explícita de cuentas autorizadas.

### RF-02

**Procesamiento inteligente de planillas utilizando técnicas de (Fuzzy Matching).**
El sistema debe permitir cargar planillas de pago (.xlsx, .csv, .txt) y auto-detectar las columnas necesarias mediante búsqueda difusa de palabras clave para CBU, Importe, Nombre y CUIT del beneficiario.
### RF-03

**Previsualización de errores antes de la generación definitiva.**
Antes de compilar el archivodefinitivo, el sistema debe desplegar una interfaz de previsualización que exponga alertas visuales claras si se detectan fallas contables (ej. CBUs que no tengan 22 dígitos numéricos). Las filas con error no formarán parte de la exportación final.

### RF-04

**Generación automática de archivos TEF de 240 caracteres.**
El portal debe transformar el listado validado en un archivo de textode ancho fijo (estándar de 240 caracteres) estructurando la cabecera (Línea U) y el detalle (Línea M) según las especificaciones de Interbanking.

### RF-05

**Sincronización automática de extractos bancarios.**
El sistema debe conectarse de forma segura a Interbanking, descargar los movimientos diarios de una cuenta mapeada e impactarlos automáticamente en el libro de bancos del ERP SAP.

### RF-06

**Actualización masiva de CBU de proveedores desde Interbanking hacia SAP.**
El sistema debe contrastar un padrón bancario de cuentas de Interbanking con el maestro de SAP. Debe categorizar el análisis en actualizaciones inmediatas, sugerencias por nombre y registros únicos, permitiendo actualizar los datos bancarios del ERP con un solo clic.

---

# Requisitos No Funcionales (RNF)

### RNF-01

**Procesamiento asíncrono de tareas pesadas.**
El procesamiento de generación de reportes y exportación de PDFs debe ocurrir en un proceso separado (CLI Background Process). El portal web debe liberar la conexión del navegador del usuario en menos de100milisegundos tras enviar la confirmación.

### RNF-02

**Sistema centralizado de logs.**
Todo error del sistema, fallo de las APIs o caída de la conexión de red del ERP debe ser capturado de forma obligatoria en un archivo físico local centralizado en la ruta storage/logs/worker.log para la auditoría de IT.

### RNF-03

**Notificaciones automáticas vía WhatsApp.**
Al completarse un impacto contable en SAP o la generación de un archivo TEF, el portal despachará de forma automática alertas en tiempo real al teléfono del Gerente de Finanzas, adjuntando el documento PDF de control.

### RNF-04

**Arquitectura modular desarrollada en PHP 8.3.**
La aplicación debe ser desarrollada de forma modular con PHP 8.3 puro, sin depender de frameworks monolíticos pesados, garantizando un despliegue inmediato en servidores Linux convencionales VPS.

---

# Historias de Usuario

## Historia de Usuario 1

### Generación de Transferencias Masivas

**COMO** Operador de Tesorería

**QUIERO** cargar una planilla de pagos y previsualizar posibles errores

**PARA** generar un archivo TEF válido para Interbanking

### Criterios de aceptación

* Validación automática de CBU y CUIT.
* Identificación visual de errores.
* Exclusión automática de registros inválidos.
* Generación de archivo plano de 240 caracteres.

---

## Historia de Usuario 2

### Sincronización de Extractos Bancarios

**COMO** Analista Contable

**QUIERO** sincronizar movimientos bancarios desde el portal

**PARA** impactarlos automáticamente en SAP Business One

### Criterios de aceptación

* Autenticación previa en SAP.
* Registro automático en BankPages.
* Reporte final de resultados.

---

# Caso de Uso Principal

## Conciliación de Extractos e Impacto Contable Directo en ERP

### Actor Principal

Analista Contable / Conciliador

### Precondiciones

* Usuario autenticado.
* Cuenta bancaria configurada.
* CBU asociado correctamente.

### Flujo Principal

| Paso | Acción                                  |
| ---- | --------------------------------------- |
| 1    | Selección de sociedad y cuenta          |
| 2    | Consulta de movimientos en Interbanking |
| 3    | Verificación de conexión SAP            |
| 4    | Mapeo de cuentas                        |
| 5    | Impacto contable                        |
| 6    | Registro en MySQL                       |
| 7    | Generación de PDF y envío por WhatsApp  |

### Flujo Alternativo

Si SAP Business One no responde:

* Se cancela el proceso.
* Se evita la duplicación de registros.
* Se informa el error al usuario.

---

# Modelo Entidad-Relación

## Tabla: bank_accounts

| Campo             | Tipo         | Descripción         |
| ----------------- | ------------ | ------------------- |
| id                | INT          | Identificador       |
| company           | ENUM         | Sociedad            |
| bank_name         | VARCHAR(100) | Banco               |
| ib_account_number | VARCHAR(50)  | Cuenta Interbanking |
| cbu               | VARCHAR(22)  | CBU                 |
| sap_account_code  | VARCHAR(50)  | Cuenta SAP          |
| is_active         | TINYINT      | Estado              |

---

## Tabla: sync_logs

| Campo                | Tipo          |
| -------------------- | ------------- |
| id                   | INT           |
| company              | VARCHAR(50)   |
| ib_account           | VARCHAR(100)  |
| sap_account          | VARCHAR(100)  |
| voucher_number       | VARCHAR(100)  |
| amount               | DECIMAL(18,2) |
| status               | VARCHAR(20)   |
| sap_journal_entry_id | INT           |
| error_message        | TEXT          |

---

## Tabla: tef_logs

| Campo         | Tipo          |
| ------------- | ------------- |
| id            | INT           |
| user_id       | VARCHAR(255)  |
| company       | VARCHAR(10)   |
| ib_account    | VARCHAR(50)   |
| transfer_type | VARCHAR(50)   |
| payment_date  | DATE          |
| row_count     | INT           |
| total_amount  | DECIMAL(15,2) |
| status        | ENUM          |
| payload       | LONGTEXT      |
| file_content  | LONGTEXT      |
| filename      | VARCHAR(255)  |

---

## Tabla: sap_sync_logs

| Campo         | Tipo         |
| ------------- | ------------ |
| id            | INT          |
| user_id       | VARCHAR(255) |
| company       | VARCHAR(10)  |
| sap_code      | VARCHAR(50)  |
| vendor_name   | VARCHAR(255) |
| old_cbu       | VARCHAR(22)  |
| new_cbu       | VARCHAR(22)  |
| banco         | VARCHAR(100) |
| nro_cuenta    | VARCHAR(50)  |
| match_type    | ENUM         |
| status        | ENUM         |
| error_message | TEXT         |

---

# Referencias y Fuentes Bibliográficas

1. PHP Association - PSR-4 Standard Specification.
2. Interbanking API Corporativa.
3. SAP Business One Service Layer Reference Guide.
4. Google Identity Platform Documentation.
