# BrandHub-Testing-Demo
Framework de automatización E2E para BrandHub | Cypress | [Demo Portafolio].

---
Diseñé e implementé una suite de automatización E2E con Cypress para BrandHub, una plataforma de branding con integraciones a modelos de IA generativa.

La solución fue integrada en Azure DevOps para ejecutar Build Validation Testing (BVT) dentro del pipeline y preparada para validaciones Post-Deployment. 
Las suites pueden ejecutarse en entornos Local, Development y Production mediante scripts organizados según el tipo de cobertura requerida.

---

### Proyecto 
Automatización E2E para plataforma de branding con IA generativa

### Objetivo 
Implementar suites de automatización E2E orientadas a Build Validation Testing (BVT) para integrarse en pipelines de CI/CD y reducir el esfuerzo de validación manual previo al deployment.

---


Stack:
```
- Cypress
- JavaScript
- Mochawesome Reporter
- dotenv-cli
- Azure Blob Storage
- Azure Static Web Apps
- Azure CLI
- GitHub
```

Mi participación
```
→ Diseño de pruebas E2E para flujos críticos de negocio.
→ Desarrollo de specs y scripts utilizando Cypress y JavaScript.
→ Incorporación de selectores en front end orientados a automatización.
→ Gestión de datos dinámicos de prueba.
→ Mantenimiento y optimización de suites.
→ Debugging, análisis de fallos y mejora continua de la confiabilidad de la suite.
→ Implementación del framework e integración con pipelines de Azure DevOps.

```

## Logros y Resultados:
- 36 escenarios automatizados.
- Reducción del tiempo de validación de aproximadamente 45 min a un promedio de 5m 10s (~89%).
- Compatibilidad con 3 entornos de ejecución.
- 7 desafíos técnicos resueltos.
- Estabilización de pruebas y validaciones pre-deployment.
- Reducción del esfuerzo de validación manual previo al deployment.
- Generación automática de reportes como artifacts en Azure.

---

## Alcance de la Solución

| Suite                | Pruebas Cubiertas                                                                                                                                                                                                                                                 |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Smoke Tests**      | Login validation<br>Signup validation                                                                                                                                                                                                                         |
| **Negative Tests**   | Invalid credentials<br>Empty user validation<br>Invalid email format validation<br>Short password validation<br>Required field validation                                                                                                                     |
| **Regression Tests** | Login regression validation & UI<br>Signup regression validation & UI<br>Questionnaire flow validation<br>Password requirement validation<br>Terms and Conditions validation<br>Privacy Policy validation<br>External links validation<br>New tab navigation validation |

--- 

## Estructura de carpetas.

```txt
brandhub-tests/
│
├── cypress/
│   ├── e2e/
│   │   ├── 1-smoke/
│   │   │   ├── login.cy.js
│   │   │   └── signup.cy.js
│   │   │
│   │   ├── 2-negative/
│   │   │   ├── login-negative.cy.js
│   │   │   └── signup-negative.cy.js
│   │   │
│   │   └── 3-regression/
│   │       ├── login-reg.cy.js
│   │       ├── questionnaire.cy.js
│   │       └── signup-reg.cy.js
│   │
│   ├── fixtures/
│   │   └── bh-credentials.json
│   │
│   ├── reports/
│   │
│   └── support/
│       ├── commands.js
│       └── e2e.js
│
├── cypress.config.js
├── package.json
├── package-lock.json
```

---

# Test Organization

La suite de automatización está organizada en ámbitos de ejecución independientes:

| Folder         | Propósito                                  |
| -------------- | ------------------------------------------ |
| `1-smoke`      | Validaciones del happy-path inicial        |
| `2-negative`   | Manejo de errores                          |
| `3-regression` | Cobertura extendida de regresión           |


---

## Organización de scripts disponibles para debugging y ejecución independiente:

| Script                  | Description                                |
| ----------------------- | ------------------------------------------ |
| `cy:smoke`              | Executes smoke suite in development        |
| `cy:smoke-prod`         | Executes smoke suite in production         |
| `cy:negative`           | Executes negative test suite               |
| `cy:negative-prod`      | Executes negative suite in production      |
| `cy:regression`         | Executes regression suite                  |
| `cy:regression-prod`    | Executes regression suite in production    |
| `cy:questionnaire`      | Executes questionnaire regression flow     |
| `cy:questionnaire-prod` | Executes questionnaire flow in production  |
| `cy:dev`                | Opens the debug UI in the dev environment  |
| `cy:local`              | Opens the debug UI in the local environment|

---

<sub>────────────────────────────────────────────────────────</sub>

# Desafíos Técnicos y Soluciones Implementadas

- Cold Starts - Backend Wake Up Calls
- Falta de selectores que ocasionan Flaky Tests
- Elementos asíncronos
- Flujo continuo del cuestionario
- Timeouts por IA generativa
- Conflictos de duplicación de usuarios
- Reportes en Cypress y artifacts en Azure

<sub>────────────────────────────────────────────────────────</sub>
  
### Problema 1:
- El backend entra en estado de suspensión tras un tiempo indeterminado que suele rondar las 48 horas.

### Solución:
Implementé una estrategia que hace un ping a la API, que "despierta" el backend antes de cualquier prueba para evitar falsos positivos;
después de validar con el equipo de desarrollo que éste error es un comportamiento esperado dada la arquitectura del servidor.

Ejemplo:

```js
describe("Login Flow - Brandhub", () => {
  before(() => {
    const API_URL = Cypress.env("API_URL");
    cy.log(`API URL: ${API_URL}`);
    cy.request({
      method: "POST",
      url: API_URL + "/api/[Described for privacy]",
      failOnStatusCode: false,
      body: {
        username: "invalid@yopmail.com",
        password: "invalidpass",
        secured: false,
      },
    }).then((response) => {
      cy.log(`Wake-up request status: ${response.status}`);
      expect(response.status).to.be.oneOf([200, 400, 401]);
    });
```
<sub>────────────────────────────────────────────────────────</sub>

### Problema 2:

- Múltiples elementos carecían de selectores definidos ocasionando tests frágiles.

### Solución

- Agregué selectores tipo ```data-cy="value"``` en el front para mejorar la confiabilidad y estabilidad de las pruebas.

  <img width="1180" height="427" alt="image" src="https://github.com/user-attachments/assets/cba89f29-43cf-490f-b6b8-61d2b8b2dd6a" />

<sub>────────────────────────────────────────────────────────</sub>

### Problema 3:

- Elementos cargaban de forma asíncrona ocasionando Flaky Tests.

### Solución

Inicialmente se utilizó un workaround para mantener la estabilidad de la prueba. Posteriormente identifiqué que ciertos elementos eran renderizados fuera del DOM, provocando fallos intermitentes. 
Tras corregir este comportamiento con el equipo de desarrollo, fue posible incorporar selectores dedicados y estabilizar la automatización.
La corrección también mejoró la consistencia de la experiencia de usuario dentro de la plataforma.

<sub>────────────────────────────────────────────────────────</sub>

### Problema 4:

El flujo del cuestionario depende de una sesión persistente, URLs generadas dinámicamente y la continuidad del estado del usuario. 
Dividir el proceso en múltiples pruebas provocaba pérdida de contexto, sesiones inválidas y creación involuntaria de usuarios adicionales.

### Solución

Se implementó como una única prueba continua capaz de realizar el registro y validar secuencialmente cada pregunta, entrada y resultado esperado. 
Esto garantiza la continuidad del flujo end-to-end y evita problemas derivados de la pérdida de estado entre pruebas.

<sub>────────────────────────────────────────────────────────</sub>

### Problema 5:

- Se actualizó el modelo de IA Generativa para la generación de nombres en el cuestionario, lo que ocasiona un timeout más largo del esperado por Cypress

### Solución

- Ajusté el ```assert``` con un ```intercept``` en Cypress que permite esperar la respuesta de la API en la pregunta específica, junto con un timeout extendido que previene el fallo del test antes de que la respuesta sea recibida.

```js
cy.intercept("POST", "[Described for privacy]").as("generateContent");

cy.get('[data-testid="submit"]').click();

cy.wait("@generateContent", { timeout: 55000 }).then((interception) => {
expect(interception.response?.statusCode).to.eq(200);
});

cy.contains(
"Here are three name suggestions based on your brand description:",  { timeout: 10000 },
).should("be.visible");
```

<sub>────────────────────────────────────────────────────────</sub>

### Problema 6:

- Flujo de registro requiere usuarios dinámicos que no se dupliquen; en el test se deben escribir siempre usuarios distintos para el registro.

### Solución

- Para los flujos de registro y creación de cuentas, el framework genera dinámicamente correos electrónicos y contraseñas durante la ejecución para evitar conflictos causados por usuarios duplicados.

Ejemplo:

```js
const email = `testuser${Date.now()}@yopmail.com`;
```

<sub>────────────────────────────────────────────────────────</sub>

### Problema 7:

- La generación y carga de reportes al contenedor de Azure solo ocurría cuando las pruebas se ejecutaban mediante scripts específicos como `npm run cy:smoke`.
- El diseño del pipeline requería la ejecución de la suite completa para validar el deployment.

### Solución

- Se ajustó la configuración del pipeline para que la máquina virtual de Ubuntu generara y publicara automáticamente los reportes como artifacts accesibles desde Azure DevOps, independientemente del tipo de ejecución utilizada.



<img width="1415" height="445" alt="image" src="https://github.com/user-attachments/assets/467788fc-8fde-4cff-a2b9-66193a401c97" />

<img width="1009" height="833" alt="image" src="https://github.com/user-attachments/assets/f6f50d33-d22b-4025-ab51-7038822769b3" />


<sub>────────────────────────────────────────────────────────</sub>


## Pipeline.

- La suite se integra con Azure DevOps para ejecutar validaciones automáticas antes del deployment y generar artifacts accesibles desde el pipeline.

<img width="1223" height="490" alt="image" src="https://github.com/user-attachments/assets/3fc4bbd1-7e0e-4a3d-aed9-6e4eb869e06b" />

<sub>────────────────────────────────────────────────────────</sub>

## Reporte HTML.

Los resultados de ejecución se generan automáticamente en formato HTML y JSON para facilitar el análisis de resultados y seguimiento de ejecuciones.
<img width="1843" height="890" alt="image" src="https://github.com/user-attachments/assets/98b9d7e9-4961-4405-93a8-a13ad3a6816e" />

<sub>────────────────────────────────────────────────────────</sub>

## Conclusión

La implementación de esta suite permitió automatizar validaciones críticas del producto mediante pruebas E2E organizadas por niveles de cobertura (Smoke, Negative y Regression), integradas dentro del proceso de CI/CD.

Además de reducir el esfuerzo de validación manual previo al deployment, la solución resolvió diversos desafíos técnicos relacionados con estabilidad, manejo de datos dinámicos, elementos asíncronos, generación de contenido mediante IA y ejecución en múltiples entornos.

La arquitectura del framework fue diseñada para ser mantenible y adaptable, permitiendo su adaptación a distintos modelos de CI/CD mediante ajustes en la configuración YAML y la incorporación de etapas adicionales de validación cuando el proceso lo requiera.

<sub>────────────────────────────────────────────────────────</sub>

## Ejemplos de integración

| Modelo                         | Flujo                                              |
| ------------------------------ | -------------------------------------------------- |
| Build Validation Testing (BVT) | Build → Test → Deploy                              |
| Post-Deployment Testing        | Build → Deploy → Test                              |
| Multi-Environment Validation   | Build Dev → Deploy Dev → Test → Deploy Prod → Test |
| Extended Validation Pipeline   | Build → Test → Deploy → Test Prod → Final Report   |


