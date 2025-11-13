# Guía para Publicar tu Página Web en GitHub Pages

## Pasos para Habilitar GitHub Pages

### 1. Sube tus archivos a GitHub (si aún no lo has hecho)

Si tus archivos locales están sincronizados con GitHub, puedes saltar este paso. Si no:

```bash
git add .
git commit -m "Subir archivos para GitHub Pages"
git push origin main
```

### 2. Configura GitHub Pages desde la Interfaz Web

1. Ve a tu repositorio en GitHub: https://github.com/estebanes/Motoro-demo

2. Haz clic en **Settings** (Configuración) en el menú superior del repositorio

3. En el menú lateral izquierdo, busca y haz clic en **Pages** (en la sección "Code and automation")

4. En la sección **Source** (Origen):
   - Selecciona **Deploy from a branch** (Desplegar desde una rama)
   - En **Branch**, selecciona **main** (o **master** si es tu rama principal)
   - En **Folder**, selecciona **/ (root)** (raíz del repositorio)
   - Haz clic en **Save** (Guardar)

5. Espera unos minutos (1-2 minutos normalmente) mientras GitHub construye tu sitio

6. Una vez que esté listo, verás un mensaje verde con la URL de tu sitio. Será algo como:
   ```
   https://estebanes.github.io/Motoro-demo/
   ```

### 3. Accede a tu Sitio Web

- Tu sitio estará disponible en: `https://estebanes.github.io/Motoro-demo/`
- GitHub Pages actualiza automáticamente tu sitio cada vez que haces un `git push` a la rama main

### 4. Verificar que index.html esté en la Raíz

**IMPORTANTE:** Asegúrate de que tu archivo `index.html` esté en la raíz del repositorio (no dentro de una carpeta). GitHub Pages buscará este archivo como la página principal.

### Notas Importantes

- Los cambios pueden tardar unos minutos en aparecer después de hacer push
- GitHub Pages es gratuito para repositorios públicos
- Si tu repositorio es privado, necesitarás una cuenta de GitHub Pro para usar GitHub Pages (o hacerlo público)
- La URL será: `https://[tu-usuario].github.io/[nombre-del-repositorio]/`

## Solución de Problemas

### Si tu sitio no aparece:
1. Verifica que hayas seleccionado la rama correcta en la configuración de Pages
2. Asegúrate de que `index.html` esté en la raíz del repositorio
3. Espera 5-10 minutos y recarga la página
4. Verifica que no haya errores en la pestaña "Actions" de tu repositorio

### Si necesitas cambiar la URL:
- Puedes cambiar el nombre del repositorio en Settings > General > Repository name
- La nueva URL será: `https://[tu-usuario].github.io/[nuevo-nombre]/`

¡Listo! Tu juego estará disponible en línea para que cualquiera pueda verlo.
