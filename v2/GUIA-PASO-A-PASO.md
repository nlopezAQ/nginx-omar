# Guía Paso a Paso — Nivel 1
## NGINX Plus + NAP WAF con GitHub Actions y Terraform Cloud

> **Para quién es esta guía:** Alguien que nunca ha usado estas herramientas antes.  
> Cada paso indica exactamente qué abrir, dónde hacer clic y qué escribir.  
> Tiempo estimado para tener el primer workflow funcionando: **~2 horas de setup + 15 min de deploy**.

---

## Índice

1. [Crear cuenta de GitHub](#1-crear-cuenta-de-github)
2. [Hacer fork del repositorio](#2-hacer-fork-del-repositorio)
3. [Crear cuenta de Terraform Cloud](#3-crear-cuenta-de-terraform-cloud)
4. [Obtener licencia de NGINX Plus (trial)](#4-obtener-licencia-de-nginx-plus-trial)
5. [Crear cuenta y credenciales de AWS](#5-crear-cuenta-y-credenciales-de-aws)
6. [Obtener API key de OMDb](#6-obtener-api-key-de-omdb)
7. [Obtener API key de TMDB](#7-obtener-api-key-de-tmdb)
8. [Generar tu par de claves SSH](#8-generar-tu-par-de-claves-ssh)
9. [Configurar los Secrets en GitHub](#9-configurar-los-secrets-en-github)
10. [Ejecutar el primer workflow](#10-ejecutar-el-primer-workflow)
11. [Verificar que funciona](#11-verificar-que-funciona)
12. [Destruir la infraestructura](#12-destruir-la-infraestructura)
13. [Solución de problemas frecuentes](#13-solución-de-problemas-frecuentes)

---

## 1. Crear cuenta de GitHub

> Si ya tienes cuenta de GitHub, salta al [paso 2](#2-hacer-fork-del-repositorio).

1. Abre tu navegador y ve a **https://github.com**
2. Haz clic en el botón verde **"Sign up"** (esquina superior derecha)
3. Ingresa tu correo electrónico → clic en **"Continue"**
4. Crea una contraseña → clic en **"Continue"**
5. Elige un nombre de usuario (por ejemplo `tu-nombre-devops`) → clic en **"Continue"**
6. Resuelve el captcha de verificación
7. Revisa tu correo → abre el email de GitHub → copia el código de 6 dígitos → pégalo en la página
8. Selecciona el plan **"Free"** → clic en **"Continue for free"**

✅ Ya tienes cuenta de GitHub.

---

## 2. Hacer fork del repositorio

Un **fork** es una copia del repositorio original en tu propia cuenta. Necesitas esto para poder configurar tus propios secrets y ejecutar tus propios workflows.

1. Ve a **https://github.com/nlopezAQ/nginx-omar**
2. En la parte superior derecha de la página, haz clic en el botón **"Fork"**
3. Se abre un formulario:
   - **Owner:** selecciona tu cuenta de GitHub
   - **Repository name:** déjalo como `nginx-omar` (o cámbialo si quieres)
   - Deja el resto como está
4. Haz clic en el botón verde **"Create fork"**
5. GitHub te lleva automáticamente a `https://github.com/TU-USUARIO/nginx-omar`

✅ El repositorio ahora es tuyo. Guarda esta URL, la usarás en todo momento.

---

## 3. Crear cuenta de Terraform Cloud

Terraform Cloud gestiona el estado de la infraestructura (qué servidores existen, qué IPs tienen, etc.). Es **gratuito** para uso personal y equipos pequeños.

1. Abre **https://app.terraform.io**
2. Haz clic en **"Create account"**
3. Ingresa:
   - **Username:** algo como `tu-nombre-devops`
   - **Email:** tu correo
   - **Password:** una contraseña segura
4. Clic en **"Create account"**
5. Revisa tu correo → abre el email de HashiCorp → clic en **"Confirm email address"**
6. Inicia sesión en **https://app.terraform.io**

### 3.1 — Crear una organización

Terraform Cloud organiza el trabajo en **organizaciones**.

1. Después de hacer login, si te pregunta "Create a new organization", escribe un nombre, por ejemplo `nginx-demo-org`
2. Si no te lo pregunta automáticamente: haz clic en el menú de usuario (esquina superior derecha) → **"Create organization"**
3. **Organization name:** escribe `nginx-demo-org` (o el nombre que quieras, sin espacios, solo letras, números y guión)
4. **Email:** deja el tuyo
5. Clic en **"Create organization"**

> 📝 **Anota este nombre de organización.** Lo necesitarás más adelante como el secret `TFC_ORG`.

### 3.2 — Crear un API Token

El API Token es la contraseña que usa GitHub Actions para conectarse a Terraform Cloud.

1. En Terraform Cloud, haz clic en tu **foto de perfil** (esquina superior derecha)
2. Clic en **"Account settings"**
3. En el menú izquierdo, clic en **"Tokens"**
4. Clic en el botón **"Create an API token"**
5. **Description:** escribe `github-actions`
6. **Expiration:** selecciona **"No expiration"** (para que no te quede sin funcionar)
7. Clic en **"Generate token"**
8. Aparece el token — una cadena larga como `atTv3.XXXXXX...`

> 🚨 **IMPORTANTE:** Este token **solo aparece una vez**. Cópialo inmediatamente y guárdalo en un lugar seguro (bloc de notas, gestor de contraseñas, etc.).  
> Lo necesitarás como el secret `TFC_TOKEN`.

✅ Terraform Cloud configurado.

---

## 4. Obtener licencia de NGINX Plus (trial)

NGINX Plus es software comercial de F5. Necesitas una licencia para instalarlo. F5 ofrece un **trial de 30 días gratuito**.

### 4.1 — Crear cuenta en MyF5

1. Abre **https://my.f5.com**
2. Haz clic en **"Create account"** (o **"Register"**)
3. Completa el formulario con tu nombre, empresa, correo, etc.
4. Revisa tu correo → confirma la cuenta

### 4.2 — Solicitar el trial de NGINX Plus

1. Inicia sesión en **https://my.f5.com**
2. En el menú superior, haz clic en **"Products & Services"** → **"My Products and Services"**
3. Busca el botón **"Get trial"** o ve directamente a **https://www.nginx.com/free-trial-request/**
4. Llena el formulario de trial:
   - Product: **NGINX Plus + App Protect WAF**
   - Completa nombre, empresa, país
5. Clic en **"Submit"**
6. Recibirás un email de confirmación. La licencia puede tardar **hasta 24 horas** en llegar.

### 4.3 — Descargar los archivos de licencia

Cuando llegue el email de activación:

1. Ve a **https://my.f5.com** → inicia sesión
2. Haz clic en **"Products & Services"** → **"My Products and Services"**
3. Busca tu suscripción de **NGINX Plus**
4. Haz clic en **"View subscription details"** o **"Manage"**
5. Descarga los siguientes archivos:
   - `nginx-repo.crt` — Certificado SSL
   - `nginx-repo.key` — Clave del certificado
   - `license.jwt` — Token JWT de licencia
   - `license.key` — Clave de licencia

> 📁 Guarda estos 4 archivos en una carpeta segura. Los pegarás como contenido en los GitHub Secrets.

✅ Licencia de NGINX Plus obtenida.

---

## 5. Crear cuenta y credenciales de AWS

AWS (Amazon Web Services) es donde se crearán los servidores (EC2) o el cluster Kubernetes (EKS). Necesitas una cuenta y credenciales programáticas.

> ⚠️ **Costo:** AWS no es gratis. Crear un EC2 `t3.medium` cuesta aproximadamente **$0.04/hora**. Si lo destruyes al terminar la demo, el costo total será de **menos de $1**.

### 5.1 — Crear cuenta de AWS

> Si ya tienes cuenta de AWS, salta al paso 5.2.

1. Abre **https://aws.amazon.com**
2. Clic en **"Create an AWS Account"** (esquina superior derecha)
3. Ingresa tu correo → clic en **"Verify email address"**
4. Revisa tu correo → copia el código de verificación → pégalo
5. **Root user password:** crea una contraseña segura
6. **Account type:** selecciona **"Personal"**
7. Completa la información de contacto
8. Ingresa tu **tarjeta de crédito** (AWS la requiere para crear la cuenta, pero el costo es mínimo si destruyes todo después)
9. **Identity verification:** elige SMS → ingresa tu teléfono → ingresa el código que te manden
10. **Support plan:** selecciona **"Basic support - Free"**
11. Clic en **"Complete sign up"**
12. Recibirás un email confirmando que la cuenta está activa (puede tardar unos minutos)

### 5.2 — Crear usuario IAM con acceso programático

Nunca uses las credenciales root de AWS en aplicaciones. Crea un usuario IAM específico.

1. Inicia sesión en **https://console.aws.amazon.com**
2. En el buscador de la parte superior, escribe **"IAM"** → clic en **"IAM"**
3. En el menú izquierdo, clic en **"Users"**
4. Clic en el botón **"Create user"**
5. **User name:** escribe `nginx-plus-deploy`
6. Clic en **"Next"**
7. **Permissions options:** selecciona **"Attach policies directly"**
8. En el buscador de políticas, busca **"AdministratorAccess"** → marca la casilla
   > ⚠️ Para una demo está bien. En producción, usa permisos más restrictivos.
9. Clic en **"Next"** → clic en **"Create user"**

### 5.3 — Crear Access Keys

1. En la lista de usuarios IAM, haz clic en el usuario `nginx-plus-deploy` que acabas de crear
2. Haz clic en la pestaña **"Security credentials"**
3. Baja hasta la sección **"Access keys"** → clic en **"Create access key"**
4. **Use case:** selecciona **"Application running outside AWS"**
5. Clic en **"Next"** → clic en **"Create access key"**
6. Aparecen dos valores:
   - **Access key ID:** algo como `AKIAIOSFODNN7EXAMPLE`
   - **Secret access key:** algo como `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`

> 🚨 La **Secret access key solo aparece una vez**. Descarga el archivo CSV o cópiala ahora.

> 📝 Anota también la **región de AWS** donde quieres desplegar, por ejemplo `us-east-1` (Virginia del Norte) o `us-west-2` (Oregón).

✅ Credenciales de AWS listas.

---

## 6. Obtener API key de OMDb

La aplicación de cine usa la API de OMDb para buscar películas. Es **gratuita**.

1. Abre **https://www.omdbapi.com/apikey.aspx**
2. Selecciona el plan **"FREE! (1,000 daily limit)"**
3. Ingresa tu correo
4. Clic en **"Submit"**
5. Recibirás un email con tu API key (algo como `abc12345`)
6. En el email hay un link de activación — haz clic en él para activar la key

> 📝 Guarda tu API key. La necesitarás como el secret `OMDB_API_KEY`.

✅ API key de OMDb lista.

---

## 7. Obtener API key de TMDB

La segunda aplicación usa The Movie Database. También es **gratuita**.

1. Abre **https://www.themoviedb.org**
2. Clic en **"Join TMDB"** (menú superior)
3. Completa el registro: username, email, contraseña
4. Revisa tu email → confirma la cuenta
5. Inicia sesión en **https://www.themoviedb.org**
6. Haz clic en tu **foto de perfil** (esquina superior derecha) → **"Settings"**
7. En el menú izquierdo, clic en **"API"**
8. Clic en **"Create"** (o **"Request an API key"**)
9. Selecciona **"Developer"**
10. Acepta los términos → completa el formulario de solicitud:
    - Application name: `nginx-plus-demo`
    - Application URL: `http://localhost`
    - Application summary: `Demo de NGINX Plus con NAP WAF`
11. Clic en **"Submit"**
12. En la página de API aparecerá tu **API Read Access Token** (también llamado `Bearer token`) y tu **API Key (v3 auth)**

> 📝 Copia el valor de **"API Key (v3 auth)"** — es la que necesitas como secret `TMDB_API_KEY`.

✅ API key de TMDB lista.

---

## 8. Generar tu par de claves SSH

Los workflows crean instancias EC2 y se conectan a ellas por SSH para instalar NGINX y desplegar las apps. Necesitas generar un par de claves.

### En Windows (usa PowerShell o Git Bash)

Abre **PowerShell** y ejecuta:

```powershell
ssh-keygen -t ed25519 -C "nginx-plus-deploy" -f "$env:USERPROFILE\.ssh\nginx-plus-deploy" -N ""
```

### En Mac o Linux (usa Terminal)

```bash
ssh-keygen -t ed25519 -C "nginx-plus-deploy" -f ~/.ssh/nginx-plus-deploy -N ""
```

Esto crea dos archivos:
- `nginx-plus-deploy` — la **clave privada** → secret `SSH_PRIVATE_KEY`
- `nginx-plus-deploy.pub` — la **clave pública** → secret `SSH_PUBLIC_KEY`

Para ver el contenido de los archivos:

```powershell
# Windows PowerShell
Get-Content "$env:USERPROFILE\.ssh\nginx-plus-deploy"      # clave privada
Get-Content "$env:USERPROFILE\.ssh\nginx-plus-deploy.pub"  # clave pública
```

```bash
# Mac/Linux
cat ~/.ssh/nginx-plus-deploy      # clave privada
cat ~/.ssh/nginx-plus-deploy.pub  # clave pública
```

> 📝 Copia cada contenido completo. La clave privada empieza con `-----BEGIN OPENSSH PRIVATE KEY-----`.

✅ Par de claves SSH generado.

---

## 9. Configurar los Secrets en GitHub

Ahora vas a poner todas las credenciales que recopilaste en tu repositorio de GitHub.  
GitHub las guarda cifradas y solo las usa cuando los workflows se ejecutan.

1. Ve a tu repositorio en GitHub: `https://github.com/TU-USUARIO/nginx-omar`
2. Haz clic en la pestaña **"Settings"** (la última en la barra de navegación del repo)
3. En el menú izquierdo, busca **"Secrets and variables"** → clic en **"Actions"**
4. Verás dos pestañas: **"Secrets"** y **"Variables"**

### 9.1 — Crear cada Secret

Para cada secret de la lista de abajo, haz lo siguiente:
1. Asegúrate de estar en la pestaña **"Secrets"**
2. Clic en el botón verde **"New repository secret"**
3. **Name:** escribe exactamente el nombre indicado (mayúsculas, sin espacios)
4. **Secret:** pega el valor correspondiente
5. Clic en **"Add secret"**

Repite para cada uno:

| # | Name | Qué pegar |
|---|---|---|
| 1 | `TFC_TOKEN` | El token de Terraform Cloud del paso 3.2 |
| 2 | `TFC_ORG` | El nombre de tu organización de Terraform Cloud (ej. `nginx-demo-org`) |
| 3 | `NGINX_REPO_CRT` | Contenido completo del archivo `nginx-repo.crt` (abre el archivo con un editor de texto como Notepad y copia todo) |
| 4 | `NGINX_REPO_KEY` | Contenido completo del archivo `nginx-repo.key` |
| 5 | `LICENSE_JWT` | Contenido completo del archivo `license.jwt` |
| 6 | `LICENSE_KEY` | Contenido completo del archivo `license.key` |
| 7 | `SSH_PUBLIC_KEY` | Contenido del archivo `nginx-plus-deploy.pub` (la clave pública) |
| 8 | `SSH_PRIVATE_KEY` | Contenido del archivo `nginx-plus-deploy` (la clave privada, incluye las líneas `-----BEGIN...` y `-----END...`) |
| 9 | `AWS_ACCESS_KEY_ID` | El Access key ID de AWS del paso 5.3 (ej. `AKIAIOSFODNN7EXAMPLE`) |
| 10 | `AWS_SECRET_ACCESS_KEY` | La Secret access key de AWS del paso 5.3 |
| 11 | `AWS_REGION` | La región de AWS donde quieres desplegar (ej. `us-east-1`) |
| 12 | `OMDB_API_KEY` | Tu API key de OMDb del paso 6 |
| 13 | `TMDB_API_KEY` | Tu API key de TMDB del paso 7 |
| 14 | `DATA_PLANE_KEY` | *(Opcional)* Token del agente NGINX One Console. Si no tienes cuenta F5 XC, déjalo vacío — el workflow continuará sin él |

> 📋 **Tip para pegar archivos .crt/.key:** Abre el archivo en Notepad (Windows) o TextEdit (Mac).  
> Selecciona todo (`Ctrl+A` o `Cmd+A`), copia (`Ctrl+C`), y pega en el campo Secret de GitHub.

Cuando termines, deberías ver 13-14 secrets en la lista.

### 9.2 — Confirmar permisos de GitHub Actions

1. En **Settings** → menú izquierdo → **"Actions"** → **"General"**
2. En "Actions permissions", asegúrate de que esté seleccionado **"Allow all actions and reusable workflows"**
3. En "Workflow permissions", selecciona **"Read and write permissions"**
4. Haz clic en **"Save"**

✅ GitHub Secrets configurados.

---

## 10. Ejecutar el primer workflow

Vamos a ejecutar el proyecto más simple: **NGINX Plus + NAP WAF v4 en dos instancias EC2**.

El workflow:
- Crea la infraestructura AWS con Terraform (VPC, security groups, 2 instancias EC2)
- Instala NGINX Plus + NAP WAF en la primera instancia
- Instala las dos apps Node.js en la segunda instancia
- Verifica que todo funciona

1. Ve a tu repositorio: `https://github.com/TU-USUARIO/nginx-omar`
2. Haz clic en la pestaña **"Actions"** (en la barra de navegación superior del repo)
3. Si aparece un banner preguntando "I understand my workflows, go ahead and enable them" → haz clic en él
4. En el menú izquierdo verás la lista de workflows
5. Busca y haz clic en **"Deploy Nginx Plus - WAF - Nginx One Agent en una VM en AWS"**
6. A la derecha aparece el botón **"Run workflow"** → haz clic en él
7. Aparece un dropdown con una sola opción → haz clic en el botón verde **"Run workflow"**

El workflow empieza a ejecutarse. Verás un punto amarillo girando que indica que está en progreso.

### ¿Qué hace cada job?

Haz clic en el workflow en ejecución para ver los detalles:

| Job | Qué hace | Tiempo aprox. |
|---|---|---|
| `setup` | Crea el workspace en Terraform Cloud | ~30 seg |
| `terraform-plan` | Planifica qué infraestructura crear | ~1 min |
| `terraform-apply` | Crea VPC + Security Groups + 2 EC2 | ~5 min |
| `install-nginx` | Instala NGINX Plus + NAP WAF en EC2-1 | ~4 min |
| `deploy-cine` | Instala Node.js + apps en EC2-2 | ~3 min |
| `configure-nginx` | Configura proxy + WAF policy | ~1 min |

Total: ~15 minutos.

✅ Cuando todos los checkmarks estén **verdes**, el deploy terminó exitosamente.

---

## 11. Verificar que funciona

1. Haz clic en el job **"deploy-cine"** en el workflow terminado
2. Expande el paso **"Display Cine app URL"** o **"Verify deployment"**
3. Verás algo como:
   ```
   ✅ Cine app deployed successfully!
   🌐 Access the app at: http://3.82.45.123:3000/
   🌐 Access the TMDB app at: http://3.82.45.123:3001/
   ```
4. Copia esa IP y abre en tu navegador: `http://3.82.45.123:3000/`
5. Deberías ver la aplicación de películas funcionando

### Probar el WAF (opcional)

Para verificar que el WAF bloquea ataques, prueba desde tu navegador o terminal:

```bash
# Intento de SQL injection — debe ser bloqueado
curl "http://3.82.45.123:3000/?id=1%27%20OR%20%271%27=%271"

# Intento de XSS — debe ser bloqueado
curl "http://3.82.45.123:3000/?q=<script>alert(1)</script>"
```

El WAF debe devolver una respuesta de bloqueo (HTTP 200 con página de bloqueo de NGINX).

✅ La aplicación está funcionando y el WAF está activo.

---

## 12. Destruir la infraestructura

> ⚠️ **Importante:** Si no destruyes la infraestructura, AWS te seguirá cobrando.  
> Destruye siempre cuando termines la demo.

1. Ve a **Actions** en tu repositorio
2. En el menú izquierdo, busca **"Destroy Nginx Plus EC2"**
3. Clic en **"Run workflow"** → **"Run workflow"**
4. Espera ~5 minutos

El workflow elimina todas las instancias EC2, la VPC, los security groups y todos los recursos de AWS creados. El workspace de Terraform Cloud se conserva (no incurre costo).

✅ Infraestructura destruida. No habrá más cargos en AWS.

---

## 13. Solución de problemas frecuentes

### El workflow falla en `terraform-apply` con "Error: Workspace not found"

**Causa:** El workspace de Terraform Cloud no se creó correctamente.  
**Solución:**
1. Ve a [app.terraform.io](https://app.terraform.io) → tu organización
2. Si el workspace `nginx-plus-one-vm` no existe, elimina el workspace si existe a medias
3. Vuelve a correr el workflow desde el principio

---

### El workflow falla con "Error: NGINX repo certificate invalid"

**Causa:** El secret `NGINX_REPO_CRT` o `NGINX_REPO_KEY` no tiene el contenido correcto.  
**Solución:**
1. Abre el archivo `.crt` con Notepad y verifica que empiece con `-----BEGIN CERTIFICATE-----`
2. Ve a **Settings** → **Secrets** → edita `NGINX_REPO_CRT` y vuelve a pegar el contenido completo
3. Asegúrate de no haber dejado espacios o caracteres extraños al principio o al final

---

### El workflow falla con "exit code 1" en el job `install-nginx`

**Causa:** La licencia de NGINX Plus expiró o las credenciales son incorrectas.  
**Solución:**
1. Ve a [my.f5.com](https://my.f5.com) → verifica que tu trial sigue vigente
2. Descarga los archivos de licencia de nuevo
3. Actualiza los secrets `NGINX_REPO_CRT`, `NGINX_REPO_KEY`, `LICENSE_JWT`, `LICENSE_KEY`

---

### Puedo hacer clic en Run Workflow pero el workflow no aparece en la lista

**Causa:** Los workflows de GitHub Actions necesitan estar en la rama `main`.  
**Solución:**
1. Ve a tu repositorio → pestaña **"Code"**
2. Verifica que la rama activa (selector de ramas, esquina superior izquierda) sea **"main"**
3. Si es otra rama, cambia a `main`
4. Ve a **Actions** de nuevo

---

### El job `setup` falla con "Unauthorized" o "403"

**Causa:** El secret `TFC_TOKEN` es incorrecto o expiró.  
**Solución:**
1. Ve a [app.terraform.io](https://app.terraform.io) → Account settings → Tokens
2. Elimina el token antiguo → crea uno nuevo
3. Ve a GitHub → Settings → Secrets → actualiza `TFC_TOKEN`

---

### No veo la URL de la app al final del workflow

**Causa:** El job `deploy-cine` terminó pero el paso de verificación puede estar dentro de un paso colapsado.  
**Solución:**
1. Haz clic en el job `deploy-cine`
2. Expande cada paso hasta encontrar uno que diga "URL" o "Access the app at"
3. Alternativamente: ve a [app.terraform.io](https://app.terraform.io) → tu organización → workspace `nginx-plus-one-vm` → **"Outputs"** → verás la IP del servidor NGINX (`nginx_public_ip`)

---

## Resumen de cuentas y datos que necesitas

| Cuenta | URL | Secret en GitHub |
|---|---|---|
| GitHub | github.com | — |
| Terraform Cloud | app.terraform.io | `TFC_TOKEN`, `TFC_ORG` |
| F5 MyF5 (NGINX Plus) | my.f5.com | `NGINX_REPO_CRT`, `NGINX_REPO_KEY`, `LICENSE_JWT`, `LICENSE_KEY` |
| AWS | console.aws.amazon.com | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION` |
| OMDb | omdbapi.com | `OMDB_API_KEY` |
| TMDB | themoviedb.org | `TMDB_API_KEY` |
| SSH key pair | generado localmente | `SSH_PUBLIC_KEY`, `SSH_PRIVATE_KEY` |

**Total: 13 secrets a configurar** (más `DATA_PLANE_KEY` si tienes NGINX One Console).

---

*Guía generada para el repositorio `nlopezAQ/nginx-omar` — Proyecto nginx-plus-one*
