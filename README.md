# Chambi: Plataforma de Geolocalización para Conectar Clientes y Expertos

## Introducción

**Chambi** es un sistema de información geográfica (SIG) diseñado para conectar a los habitantes de Cali, Colombia, con expertos en servicios domiciliarios y profesionales. Esta plataforma digital busca centralizar la oferta y demanda de servicios, empleando geolocalización para agilizar las interacciones entre clientes y expertos.

Inspirada en el éxito de plataformas como TaskRabbit y Thumbtack, **Chambi** garantiza transparencia, confianza y seguridad en las transacciones, promoviendo la formalización laboral y el crecimiento del sector servicios local.

### Problema

Actualmente, la búsqueda de expertos confiables en Cali está fragmentada en canales informales o desactualizados. Esto genera desconfianza, pérdida de tiempo y dificultades tanto para clientes como para proveedores de servicios. **Chambi** surge como respuesta a estas problemáticas, con un enfoque centrado en la eficiencia y la fiabilidad.

---

## Accede al test online

Puedes acceder atraves del siguiente vinculo: Hosting URL: https://chambi-pro.web.app

Actualmente la base de datos solo esta poblada para ejemplos con la comuna 1 y para la categoria de servicios, serivicos de "reparacion y mantenimiento" y "Reparacion de electrodomesticos"

---

## Características Clave

1. **Geolocalización Precisa**: Utiliza las coordenadas del cliente para mostrar opciones cercanas.
2. **Perfiles Detallados y Verificados**: Información sobre habilidades, certificaciones y tarifas.
3. **Búsqueda Personalizada**: Filtros avanzados para afinar los resultados por tipo de servicio, precio y disponibilidad.
4. **Interfaz Interactiva**: Incluye mapas dinámicos y listas ordenadas por proximidad.
5. **Base de Datos Robusta**: Implementación en Firebase para asegurar escalabilidad y seguridad.
6. **Seguridad y Privacidad**: Mecanismos de autenticación y manejo seguro de datos personales.

---

## Tecnologías Utilizadas

- **Frontend**: React.js con soporte para SPA.
- **Backend**: Firebase con Realtime Database.
- **Geolocalización**: API de Google Maps y cálculos propios de proximidad.
- **Servidor de Mapas**: Leaflet para visualización dinámica.
- **Hosting**: Firebase Hosting con soporte SSL.
- **Herramientas de Desarrollo**: Node.js, npm, y Firebase CLI.

---

## Instalación

### Requisitos Previos

1. Node.js v16 o superior.
2. Firebase CLI configurado.
3. Cuenta activa en Firebase.
4. API Key de Google Maps.

### Pasos de Instalación

1. **Clonar el repositorio:**

   ```bash
   git clone https://github.com/andresburbans/chambi.git
   cd chambi
   ```

2. **Instalar dependencias:**

   ```bash
   npm install
   ```

3. **Configurar Firebase:**

   - Crear un proyecto en [Firebase](https://firebase.google.com/).
   - Habilitar Realtime Database y Authentication.
   - Descargar el archivo `firebaseConfig` desde Firebase Console.
   - Guardarlo en `src/config/firebaseConfig.js` con el siguiente formato:
     ```javascript
     const firebaseConfig = {
       apiKey: "YOUR_API_KEY",
       authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
       databaseURL: "https://YOUR_PROJECT_ID.firebaseio.com",
       projectId: "YOUR_PROJECT_ID",
       storageBucket: "YOUR_PROJECT_ID.appspot.com",
       messagingSenderId: "YOUR_SENDER_ID",
       appId: "YOUR_APP_ID"
     };
     export default firebaseConfig;
     ```

4. **Configurar Google Maps:**

   - Obtener una API Key desde [Google Cloud Console](https://console.cloud.google.com/).
   - Crear un archivo `.env` en la raíz del proyecto:
     ```env
     REACT_APP_GOOGLE_MAPS_API_KEY=tu_api_key
     ```

5. **Iniciar el servidor local:**

   ```bash
   npm start
   ```

   Accede a la aplicación en `http://localhost:3000`.

---

## Estructura del Proyecto

### Directorios Principales

- **`src/components`**: Contiene componentes reutilizables como mapas, formularios y encabezados.
- **`src/pages`**: Estructura de las vistas principales como inicio, búsqueda y perfil.
- **`src/services`**: Conexiones a Firebase y API externas.
- **`src/config`**: Archivos de configuración como Firebase y variables de entorno.
- **`src/styles`**: Archivos CSS y estilos globales.

### Arquitectura General

- **Cliente**: React.js para gestionar vistas y lógica de usuario.
- **Servidor**: Firebase para autenticación, base de datos y hosting.
- **Mapas**: Leaflet con soporte para datos dinámicos y markers personalizados.

---

## Funcionalidades Específicas

### Geolocalización para Registro de Usuarios

- **Propósito**: Registrar a los usuarios junto con su ubicación geográfica para ofrecer una experiencia personalizada.
- **Implementación Técnica:**
  ```javascript
  import { getAuth, createUserWithEmailAndPassword } from "firebase/auth";
  import { getDatabase, ref, set } from "firebase/database";

  const registerUser = (email, password, location) => {
    const auth = getAuth();
    const db = getDatabase();

    createUserWithEmailAndPassword(auth, email, password)
      .then((userCredential) => {
        const userId = userCredential.user.uid;
        set(ref(db, `users/${userId}`), {
          email,
          location,
        });
      })
      .catch((error) => console.error("Error: ", error));
  };
  ```

### Búsqueda en `SearchServices.jsx`

- **Propósito**: Permitir a los clientes encontrar expertos cercanos según su ubicación.
- **Implementación Técnica:**
  ```javascript
  import React, { useState, useEffect } from "react";
  import { getDatabase, ref, onValue } from "firebase/database";
  import Map from "../components/Map";

  const SearchServices = ({ userLocation }) => {
    const [services, setServices] = useState([]);

    useEffect(() => {
      const db = getDatabase();
      const servicesRef = ref(db, 'services');

      onValue(servicesRef, (snapshot) => {
        const data = snapshot.val();
        const filteredServices = Object.values(data).filter(service => 
          calculateDistance(userLocation, service.location) < 5 // Dentro de 5 km
        );
        setServices(filteredServices);
      });
    }, [userLocation]);

    return <Map services={services} userLocation={userLocation} />;
  };

  export default SearchServices;
  ```

### Lógica de Cálculo de Distancia

Utiliza la fórmula de Haversine para determinar la distancia entre dos puntos geográficos.

```javascript
const calculateDistance = (location1, location2) => {
  const toRad = (value) => (value * Math.PI) / 180;
  const R = 6371; // Radio de la Tierra en km

  const dLat = toRad(location2.lat - location1.lat);
  const dLon = toRad(location2.lng - location1.lng);

  const a =
    Math.sin(dLat / 2) * Math.sin(dLat / 2) +
    Math.cos(toRad(location1.lat)) * Math.cos(toRad(location2.lat)) *
    Math.sin(dLon / 2) * Math.sin(dLon / 2);

  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
  return R * c; // Distancia en km
};
```

---

## Estado del Proyecto

Aunque el despliegue aún no se ha realizado, se tiene plena integración con Firebase Hosting. Esto permite realizar un despliegue rápido y fiable. Las funcionalidades están diseñadas para facilitar futuras expansiones como pagos en línea, soporte multiidioma y métricas avanzadas.

---

## Documentación Adicional

Este README se creó con base en la rúbrica "Capítulo 3 - Diseño SIG3" para garantizar un desarrollo académico y técnico alineado con las mejores prácticas de desarrollo.

---

## Contribuciones

1. Realiza un fork del repositorio.
2. Crea una rama para tus cambios.
3. Envíanos un pull request con una descripción detallada de las mejoras.

---

## Licencia

Este proyecto está bajo la Licencia MIT.

