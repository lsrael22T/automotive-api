# SPR AUTOMOTIVE API

## Introducción

SPR automotive proporciona una API tipo Rest con autenticación a través de tokens. Nuestra API permite consultar productos de nuestra base de datos con toda la información necesaria de acuerdo al nivel de acceso.

## Credenciales necesarias

Para consumir la API es necesario contar con dos tipos de credenciales:

- **Credenciales de API:** Estas credenciales son exclusivas para cada empresa / aplicación. Dichas credenciales son proporcionadas por SPR Automotive y constan de un `client_id` y una llave `client_secret`.
- **Credenciales de usuario web:** Estas credenciales sirven para identificar al usuario web que está haciendo uso de la API y constan de un `username` y un `password`. Dichas credenciales son las mismas que se le proporcionan a los usuarios finales de [sprautomotive.com](https://sprautomotive.com)

> **NOTA:** Aunque ya se cuente con credenciales de **usuario web** es necesario contar con ambos tipos de credenciales, de lo contrario no será posible consumir la información de la API.

## Métodos de autenticación

Existen dos métodos de autenticación conocidos como `grant_type`:

- **password**: lo utilizamos para iniciar sesión en el sistema con nuestras credenciales, tanto credenciales de tipo _API_ como de tipo _usuario_.
- **refresh_token**: lo utilizamos para obtener un nuevo token de autenticación sin necesidad de especificar username y el password. Para este último es necesario contar con un token tipo `password` vigente.

## Obtener token de autorización (Login)

El siguiente método sirve para obtener un token de inicio de sesión, dicho token es el que nos dará acceso a la API.

### URL:

https://sprautomotive.com/oauth/token

### Método:

POST

### Encabezados:

**Accept:** application/json

### Body (form-data):

- **grant_type:** password
- **client_id:** _id de credenciales tipo API_
- **client_secret:** _llave de credenciales tipo API_
- **username:** _usuario web y región separados por @. ejemplo: `admin@es_MX`_
- **password:** _contraseña del usuario web_

Ejemplo usando laravel php:

```php
$response = Http::asForm()
->headers([
  'accept' => 'application/json'
])
->post('https://sprautomotive.com/oauth/token', [
  'grant_type' => 'password',
  'client_id' => '25',
  'client_secret' => 'a418c019f47137fbb55b63142958a056',
  'username' => 'admin@es_MX',
  'password' => 'myVerySecurePassword123'
]);

return $response->json();
```

Si la solicitud es correcta recibiremos la siguiente respuesta:

```json
{
  "token_type": "Bearer",
  "expires_in": 43200,
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSU...",
  "refresh_token": "def502003f5d7f4bd0f08b..."
}
```

Los valores son los siguientes:

- **token_type:** Especifíca el tipo de token que hemos recuperado, en este caso siempre será de tipo `Bearer`.
- **expires_in:** Especifíca la vigencia de nuestro token en segundos, normalmente son 43200 segundos (12 horas).
- **access_token:** Este es el token que nos dará acceso a la API.
- **refresh_token**: Este token nos sirve para obtener un nuevo token de acceso en caso de que nuestro token anterior esté a punto de expirar.

En caso de recibir algún error, la respuesta tendrá la siguiente estructura:

```json
{
  "error": "invalid_grant",
  "error_description": "The user credentials were incorrect.",
  "message": "The user credentials were incorrect."
}
```

> **NOTA:** el tipo de error, la descripción y el mensaje pueden variar de acuerdo a los datos enviados en el formulario.

## Renovar token de autorización (Refresh token)

El siguiente método sirve para evitar que nuestra sesión caduque. Cuando nuestro `access_token` esté a punto de expirar podemos usar este método para obtener un nuevo token de acceso sin necesidad de volver a introducir usuario y contraseña.

### URL:

https://sprautomotive.com/oauth/token

### Método:

POST

### Encabezados:

**Accept:** application/json

### Body (form-data):

- **grant_type:** refresh_token
- **client_id:** _id de credenciales tipo API_
- **client_secret:** _llave de credenciales tipo API_
- **refresh_token:** aquí ponemos el `refresh_token` que recibimos junto al token de acceso descrito en la sección anterior.

Ejemplo usando laravel php:

```php
$response = Http::asForm()
->headers([
  'accept' => 'application/json'
])
->post('https://sprautomotive.com/oauth/token', [
  'grant_type' => 'refresh_token',
  'client_id' => '25',
  'client_secret' => 'a418c019f47137fbb55b63142958a056',
  'refresh_token' => 'def502003f5d7f4bd0f08b...',
]);

return $response->json();
```

Si la solicitud es correcta recibiremos la siguiente respuesta:

```json
{
  "token_type": "Bearer",
  "expires_in": 43200,
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...",
  "refresh_token": "def5020099e06ac2091ba9c0df79d46f9f40b..."
}
```

> NOTA: Una vez que recibimos nuevos `access_token` y `refresh_token`, los anteriores quedan completamente invalidados y ya no podrán usarse para autenticación.

## Consumir datos de la API

Una vez que contamos con nuestro token de acceso ya podremos consumir la información de la API. Para ello, toda solicitud web debe incluir los siguientes headers:

- **Accept:** application/json
- **Authorization:** Bearer <access_token>

Ejemplo usando laravel php:

```php
$response = Http::asForm()
->headers([
  'accept' => 'application/json',
  'Authorization' => 'Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...'
])
->post('https://sprautomotive.com/es_MX/api/products/DE-814');

return $response->json();
```

# Recursos disponibles

Lista de los recursos que actualmente se pueden consumir desde nuestra API. La siguiente lista se encuentra en construcción y próximamente habrá nuevos recursos.

## Producto

Obtener información especifíca de un producto conociendo su código.

### URL:

https://sprautomotive.com/{locale}/api/products/{code}

### Variables:

- **locale:** Código ISO de la región del usuario autenticado
- **code:** Código del producto a consultar

### Método:

GET

### Encabezados:

- **Accept:** application/json
- **Authorization:** token de autenticación

### Solicitud

usando laravel php:

```php
$response = Http::asForm()
->headers([
  'accept' => 'application/json',
  'Authorization' => 'Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...'
])
->get('https://sprautomotive.com/es_MX/api/products/DE-814');

return $response->json();
```

### Respuesta

- **code:**
  - **Tipo**: String
  - **Permisos:** _Ninguno_
  - **Descripción:** Código del producto
- **detailed_description:**
  - **Tipo**: String
  - **Permisos:** _Ninguno_
  - **Descripción:** Información más relevante del producto
- **apps:**
  - **Tipo**: Object
  - **Permisos:** _Lectura de aplicaciones_
  - **Descripción:** Lista de compatibilidad de vehículos con estructura de árbol
- **picture_url:**
  - **Tipo**: String
  - **Permisos:** _Ningunno_
  - **Descripción:** URL de la foto principal del producto alojada en el servidor
- **price_00:**
  - **Tipo**: Object
  - **Permisos:** _Lectura de precios, cuenta de telemarketing_
  - **Descripción:** Precio del producto de lista (precio base)
  - **Contenido:**
    - _value:_ precio del producto
    - _prefix:_ símbolo de moneda
    - _sufix:_ código ISO de moneda
- **price_01:**
  - **Tipo**: Object
  - **Permisos:** _Lectura de precios_
  - **Descripción:** Precio del producto para el cliente (usuario autenticado)
  - **Contenido:**
    - _value:_ precio del producto
    - _prefix:_ símbolo de moneda
    - _sufix:_ código ISO de moneda
- **stock:**
  - **Tipo**: Array
  - **Permisos:** _Lectura de existencias, Sucursales habilitadas_
  - **Descripción:** Lista de existencias del producto en diferentes sucursales
  - **Contenido por item:**
    - _branch_id:_ Identificador de sucursal
    - _branch_name:_ Nombre de sucursal
    - _quantity:_ Cantidad de piezas disponibles
- **category:**
  - **Tipo**: Object
  - **Permisos:** _Ninguno_
  - **Descripción:** Línea / categoría a la que pertenece el producto
  - **Contenido:**
    - _singular:_ Nombre regiónal en singular
    - _plural:_ Nombre regiónal en plural
    - _keywords:_ Palabras clave para identificar la categoría
    - _line:_ Línea a la que pertenece el producto
      - _name_short:_ Identificador corto
      - _name_long:_ Identificador descriptivo
      - _description:_ Información de productos que contiene la línea
- **props:**
  - **Tipo**: Array
  - **Permisos:** _Lectura de atributos de productos_
  - **Descripción:** Lista de atributos con los que cuenta el producto
  - **Contenido por item:**
    - _name:_ Nombre de del atributo
    - _content:_ Valor del atributo
- **oems:**
  - **Tipo**: Array
  - **Permisos:** _Lectura de oems de productos_
  - **Descripción:** Lista códigos de ensambladora
  - **Contenido por item:**
    - _code:_ Código OEM
    - _make:_ Datos del fabricante
      - _value:_ Nombre del fabricante
- **interchanges:**
  - **Tipo**: Array
  - **Permisos:** _Lectura de conversiones de productos_
  - **Descripción:** Lista conversiones / códigos alternos de productos
  - **Contenido por item:**
    - _code:_ Código alterno
    - _make:_ Datos del fabricante
      - _value:_ Nombre del fabricante

```json
{
  "code": "RT-510056",
  "apps": {
    "FORD": {
      "FIESTA": [
        {
          "make": "FORD",
          "model": "FIESTA",
          "year": "2003-2013",
          "engine": "L4 1.6L",
          "fueltype": "GAS",
          "drivetype": "FWD",
          "position": "DEL"
        }
      ],
      "FOCUS": [
        {
          "make": "FORD",
          "model": "..."
        },
        {
          "make": "FORD",
          "model": "..."
        }
      ]
    },
    "VOLKSWAGEN": {
      "GOL": [
        {
          "make": "VOLKSWAGEN",
          "model": "..."
        }
      ]
    }
  },
  "price_00": {
    "value": 12.34,
    "prefix": "$",
    "sufix": "MXN"
  },
  "price_01": {
    "value": 12.34,
    "prefix": "$",
    "sufix": "MXN"
  },
  "stock": [
    {
      "branch_id": 1,
      "branch_name": "Fresnillo, Zac",
      "quantity": 123
    },
    {
      "branch_id": 4,
      "branch_name": "Monterrey, NL",
      "quantity": 123
    }
  ],
  "category": {
    "singular": "Balero de Maza",
    "plural": "Baleros de Maza",
    "keywords": "cojinete cojinetes rodamiento rodamientos",
    "line": {
      "name_short": "rodatech",
      "name_long": "Rodatech Automotive",
      "description": "Mazas, distribución y rodamientos"
    }
  },
  "props": [
    {
      "content": "72",
      "name": "Diámetro exterior"
    }
  ],
  "oems": [
    {
      "code": "6S4Z-1215-B",
      "make": {
        "value": "FORD"
      }
    }
  ],
  "interchanges": [
    {
      "code": "XYZ123456",
      "make": {
        "value": "ANOTHERSELLER"
      }
    }
  ]
}
```

## Catálogo de Productos

Obtener lista de productos de acuerdo a los parámetros enviados.

### URL:

https://sprautomotive.com/{locale}/api/products

### Variables:

- **locale:** Código ISO de la región del usuario autenticado

### Parámetros de URL

#### Obligatorias

- **year:** Año del vehículo
- **make:** Marca del vehículo

#### Opcionales

- **model:** Modelo del vehículo
- **category:** Línea de productos
- **page:** No. de página a consultar. Aplica si la consulta contiene más de 30 productos

### Método:

GET

### Encabezados:

- **Accept:** application/json
- **Authorization:** token de autenticación

### Solicitud

usando laravel php:

```php
$response = Http::asForm()
->headers([
  'accept' => 'application/json',
  'Authorization' => 'Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...'
])
->get('https://sprautomotive.com/api/products?year=2011&make=ford&model=fiesta&category=mazas+de+rueda');

return $response->json();
```

### Respuesta

- **per_page:**
  - **Tipo**: Integer
  - **Permisos:** _Ninguno_
  - **Descripción:** Total de registros que se muestran por página, de forma predeterminada son 30
- **current_page:**
  - **Tipo**: Integer
  - **Permisos:** _Ninguno_
  - **Descripción:** Página de consulta actual
- **from:**
  - **Tipo**: Integer
  - **Permisos:** _Ninguno_
  - **Descripción:** Número de registro en el que comienza la lista de la página actual
- **to:**
  - **Tipo**: Integer
  - **Permisos:** _Ninguno_
  - **Descripción:** Número de registro en el que termina la lista de la página actual
- **data:**
  - **Tipo**: Array
  - **Permisos:** _Ninguno_
  - **Descripción:** Lista de productos que coinciden con los parámetros enviados,
  - **Contenido por item:**
    - Incluye la estructura de producto del [anteriormente](#producto), con la diferencia de que no incluye aplicaciones, equivalencias y propiedades de producto.

```json
{
  "current_page": 1,
  "data": [
    {
      "code": "RT-512490",
      "detailed_description": "MAZA DE RUEDA TRAS. FWD ABS FORD FI",
      "price_00": {
        "value": 889.85,
        "prefix": "$",
        "sufix": "MXN"
      },
      "price_01": {
        "value": 889.85,
        "prefix": "$",
        "sufix": "MXN"
      },
      "stock": [
        {
          "branch_id": 1,
          "branch_name": "Fresnillo, Zac",
          "quantity": 165
        },
        {
          "branch_id": 2,
          "branch_name": "CDMX",
          "quantity": 18
        }
      ],
      "category": {
        "singular": "Maza de Rueda",
        "plural": "Mazas de Rueda",
        "line": {
          "name_short": "rodatech",
          "name_long": "Rodatech Automotive",
          "description": "Mazas, distribución y rodamientos"
        }
      }
    }
  ],
  "from": 1,
  "to": 1,
  "per_page": 30
}
```

## Búsqueda de productos

Obtener lista de productos de acuerdo a los parámetros de búsuqeda

### URL:

https://sprautomotive.com/{locale}/api/products

### Variables:

- **locale:** Código ISO de la región del usuario autenticado

### Parámetros de URL

#### Obligatorias

- **search:** Términos de búsqueda

#### Opcionales

- **page:** No. de página a consultar. Aplica si la consulta contiene más de 30 productos

### Método:

GET

### Encabezados:

- **Accept:** application/json
- **Authorization:** token de autenticación

### Solicitud

usando laravel php:

```php
$response = Http::asForm()
->headers([
  'accept' => 'application/json',
  'Authorization' => 'Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...'
])
->get('https://sprautomotive.com/api/products?search=balatas+aveo');

return $response->json();
```

### Respuesta

- **per_page:**
  - **Tipo**: Integer
  - **Permisos:** _Ninguno_
  - **Descripción:** Total de registros que se muestran por página, de forma predeterminada son 30
- **current_page:**
  - **Tipo**: Integer
  - **Permisos:** _Ninguno_
  - **Descripción:** Página de consulta actual
- **from:**
  - **Tipo**: Integer
  - **Permisos:** _Ninguno_
  - **Descripción:** Número de registro en el que comienza la lista de la página actual
- **to:**
  - **Tipo**: Integer
  - **Permisos:** _Ninguno_
  - **Descripción:** Número de registro en el que termina la lista de la página actual
- **data:**
  - **Tipo**: Array
  - **Permisos:** _Ninguno_
  - **Descripción:** Lista de productos que coinciden con los parámetros enviados,
  - **Contenido por item:**
    - Incluye la misma estructura de [**producto**](#producto), con la diferencia de que no incluye aplicaciones, equivalencias y propiedades de producto.

```json
{
  "current_page": 1,
  "data": [
    {
      "code": "PC1269K",
      "detailed_description": "PASTILLAS DE FRENOS DEL. CHEVROLET",
      "price_00": {
        "value": 326.94,
        "prefix": "$",
        "sufix": "MXN"
      },
      "price_01": {
        "value": 326.94,
        "prefix": "$",
        "sufix": "MXN"
      },
      "stock": [
        {
          "branch_id": 1,
          "branch_name": "Fresnillo, Zac",
          "quantity": 43
        },
        {
          "branch_id": 2,
          "branch_name": "CDMX",
          "quantity": 3
        }
      ],
      "category": {
        "singular": "Pastillas de Frenos",
        "plural": "Pastillas de Frenos",
        "line": {
          "name_short": "partech",
          "name_long": "Partech",
          "description": "Autopartes hidráulicas y sistema de frenado"
        }
      }
    }
  ],
  "from": 1,
  "to": 4,
  "per_page": 30
}
```

## Inventario (Solo Guatemala)

Obtener información en tiempo real de inventario

### URL:

https://sprautomotive.com/{locale}/api/inventory?page=1&enterprise=rodatech

### Variables:

- **locale:** Código ISO de la región del usuario autenticado

### Parámetros de URL

- **page:** Página de registros que se desea consultar. Si no se especifica, el valor predeterminado es 1
- **enterprise:** Empresa que se desea consultar. Si no se especifica, el valor predeterminado es `rodatech`, otros valores: `partech`, `shark`
- **CORRELATIVODUCAING**: Opcional - No. POLIZA (INGRESO).
- **FECHAINGRECINTO**: Opcional - FECHA DE ARRIBO.
- **DESCMERCAINGRESO**: Opcional - DESCRIPCION DE LAS MERCANCIAS (INGRESO).
- **CANTBULTOSING**: Opcional - CANTIDAD DE BULTOS (INGRESO).
- **NUMLOTEEGRESO**: Opcional - No. POLIZA (EGRESO).
- **DESCMERCAENGRESO**: Opcional - DESCRIPCION DE LAS MERCANCIAS (EGRESO).
- **BULTOSEGRESO**: Opcional - CANTIDAD DE BULTOS (EGRESO).
- **INVBULTOS**: Opcional - SALDO EN INVENTARIO.

### Método:

GET

### Encabezados:

- **Accept:** application/json
- **Authorization:** token de autenticación

### Solicitud

usando laravel php:

```php
$response = Http::asForm()
->headers([
  'accept' => 'application/json',
  'Authorization' => 'Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...'
])
->get('https://sprautomotive.com/es_GT/api/inventory?page=2&enterprise=rodatech');

return $response->json();
```

### Respuesta

- **per_page:**
  - **Tipo**: Integer
  - **Permisos:** _Ninguno_
  - **Descripción:** Total de registros que se muestran por página, de forma predeterminada son 100
- **current_page:**
  - **Tipo**: Integer
  - **Permisos:** _Ninguno_
  - **Descripción:** Página de consulta actual
- **from:**
  - **Tipo**: Integer
  - **Permisos:** _Ninguno_
  - **Descripción:** Número de registro en el que comienza la lista de la página actual
- **to:**
  - **Tipo**: Integer
  - **Permisos:** _Ninguno_
  - **Descripción:** Número de registro en el que termina la lista de la página actual
- **data:**
  - **Tipo**: Array
  - **Permisos:** _Ninguno_
  - **Descripción:** Lista de registros,
  - **Contenido por item:**
    - **NOMBRECINTO**: Recinto Aduanero
    - **CORRELATIVODUCAING**: No. Póliza (Ingreso)
    - **NUMORDENDUCA**: No. DUA (Ingreso)
    - **CONSIGNATARIO**: Consignatario
    - **FECHAINGRECINTO**: Fecha de Arribo
    - **PLACAS**: Placas
    - **NUMCONTENEDOR**: No. Contenedor
    - **CARTACUPO**: Carta de Cupo
    - **DESCMERCAINGRESO**: Descripción de las Mercancías (Ingreso)
    - **CANTBULTOSING**: Cantidad de Bultos (Ingreso)
    - **NUMLOTEEGRESO**: No. Póliza (Egreso)
    - **DUA**: No. DUA (Egreso)
    - **DESCMERCAENGRESO**: Descripción de las Mercancías (Egreso)
    - **BULTOSEGRESO**: Cantidad de Bultos (Egreso)
    - **INVBULTOS**: Saldo en Inventario
    - **NUMORDEN**: No. DUA
    - **IMPUESTOS**: Impuestos (DAI - IVA)
    - **UBICACIONBODEGA**: Ubicación de las Mercancías
    - **OBSBODEGA**: Observaciones
    - **FECHAABANDONO**: Fecha que Cae en Abandono
    - **ESTATUS**: Estatus de las Mercancías

```json
{
  "current_page": 2,
  "data": [
    {
      "NOMBRECINTO": "ZZ1 - Zona Franca Parque Industrial Zeta La Union, S.A.",
      "CORRELATIVODUCAING": "SAT-ZZ1-780-2023",
      "NUMORDENDUCA": "",
      "CONSIGNATARIO": "AH GUATEMALA, SOCIEDAD ANONIMA",
      "FECHAINGRECINTO": "2023-10-31 00:00:00.000",
      "PLACAS": "",
      "NUMCONTENEDOR": "",
      "CARTACUPO": "",
      "DESCMERCAINGRESO": "111920 BOMBA DE CLUTCH",
      "CANTBULTOSING": "6.0",
      "NUMLOTEEGRESO": "327-1703359-0086-0780-23",
      "DUA": "",
      "DESCMERCAENGRESO": "111920 BOMBA DE CLUTCH",
      "BULTOSEGRESO": "0.0",
      "INVBULTOS": "6.0",
      "NUMORDEN": "",
      "IMPUESTOS": "5.7350399999999997",
      "UBICACIONBODEGA": "33-36",
      "OBSBODEGA": "BOMBA DE CLUTCH FORD ESCORT L4 1.9 91-96/MAZDA PROTEGE 90-95",
      "FECHAABANDONO": "C20",
      "ESTATUS": "C21"
    },
    ...{}
  ],
  "from": 101,
  "per_page": 100,
  "to": 200
}

```
