# Punto de Venta Cremerías Chinos

Sistema de punto de venta de escritorio para una cremería con una o varias sucursales. Funciona
**offline-first**: cada caja guarda todo en su propia base de datos local (SQLite) y, si se conecta,
sincroniza con un proyecto de Supabase compartido entre todas las sucursales.

**Versión actual: 1.0.0**

## Tecnología

- **Java 21** + **JavaFX 21** (interfaz de escritorio), tema [AtlantaFX](https://github.com/mkpaz/atlantafx) e iconos [Ikonli](https://kordamp.org/ikonli/).
- **SQLite** como base de datos local de cada caja (offline-first).
- **Supabase** (PostgreSQL + Edge Functions) como nube opcional para sincronizar varias cajas/sucursales.
- **BCrypt** para contraseñas, **SLF4J + Logback** para logs, **Jackson** para JSON.
- Empaquetado con **jpackage** (ejecutable e instalador autocontenidos, sin pedir Java instalado), para Windows y macOS.

## Módulos principales

- **Caja / Venta**: cobro con báscula (PLU), código de barras, pago mixto, ventas en espera, notas sin venta.
- **Catálogo**: productos, presentaciones (kilo, pieza, caja), categorías, precios por canal (mostrador/mayoreo), temporadas.
- **Inventario**: existencias por sucursal y almacén, lotes con PEPS (primero en entrar, primero en salir), ajustes, transformaciones, recepción de surtidos, reabastecimiento automático.
- **Compras y devoluciones**: compras a proveedor, devoluciones de cliente y a proveedor.
- **Administración**: usuarios y roles, permisos, reglas de horario de caja, gastos, nómina, crédito a clientes.
- **Indicadores**: ventas por día, evolución, reconciliación de caja (arqueo).
- **Sincronización con la nube**: una caja nueva se conecta sola la primera vez que entra un administrador de Supabase; después sincroniza en segundo plano. Ver [`supabase/README.md`](supabase/README.md) para el detalle del backend.

## Roles

`ADMINISTRADOR`, `SUPERVISOR` y `CAJERO`, cada uno con su propia vista y permisos (ver `src/main/java/com/cremerias/puntoventa/ui/roles` y `service/Permisos.java`).

## Estructura del proyecto

```
src/main/java/com/cremerias/puntoventa/
├── ui/            interfaz JavaFX (admin, supervisor, caja, componentes compartidos)
├── service/       lógica de negocio (ventas, inventario, administración, indicadores...)
├── repository/    acceso a la base de datos local (SQLite)
├── sync/          conexión y sincronización con Supabase
├── db/            conexión, migraciones y respaldos de la base local
├── model/         entidades del dominio
├── security/      hash de contraseñas
└── config/        rutas de datos de la aplicación (AppPaths)

supabase/          esquema, políticas y Edge Functions del backend en la nube
packaging/         scripts para generar el ejecutable/instalador
docs/              auditoría, plan y bitácora de cambios del proyecto
```

## Requisitos para desarrollar

- JDK 21+ (con `jpackage` incluido, viene en el JDK normal).
- Maven Wrapper incluido (`mvnw` / `mvnw.cmd`), no se necesita Maven instalado.
- El proyecto en sí es multiplataforma; `packaging/empaquetar.ps1` genera el instalador de Windows y `packaging/empaquetar-prueba-mac.sh` el de macOS.

## Correr en modo desarrollo

```
./mvnw.cmd clean javafx:run
```

## Pruebas

```
./mvnw.cmd test
```

## Generar el ejecutable / instalador

```
powershell -ExecutionPolicy Bypass -File packaging\empaquetar.ps1                  # version normal
powershell -ExecutionPolicy Bypass -File packaging\empaquetar.ps1 -Instalador      # ademas el instalador .exe (requiere WiX Toolset)
powershell -ExecutionPolicy Bypass -File packaging\empaquetar.ps1 -Prueba          # version de prueba: solo local, base limpia, admin / admin123
```

La bandera `-Prueba` arma una build que **nunca se conecta a la nube** (aunque el jar traiga el
proyecto de Supabase) y guarda sus datos en `~/.cremerias-pos-prueba`, separado de una instalación
real (`~/.cremerias-pos`). Al primer arranque solo crea el usuario `admin` / `admin123`, sin
sucursales ni catálogo de ejemplo — todo lo demás lo da de alta el propio administrador. Se usa
para que alguien pruebe la aplicación sin tocar datos reales ni necesitar internet.

En macOS se usa `packaging/empaquetar-prueba-mac.sh` para generar el `.app`/`.dmg` equivalente.

El resultado queda en `target\dist\image\<nombre>\` (carpeta portable) y, con `-Instalador`, también
en `target\dist\installer\`.
