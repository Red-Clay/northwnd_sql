# Proyecto Northwind PostgreSQL - Modificado

Este repositorio contiene una versión modificada de la base de datos Northwind para PostgreSQL, desarrollada como proyecto de curso con nuevas funcionalidades y mejoras.

## 📋 Descripción del Proyecto

La base de datos Northwind ha sido extendida con las siguientes mejoras:

### ✨ Nuevas Funcionalidades

1. **Atributos dinámicos en productos (JSON)**: 
   - Se añadió una columna JSON (`caracteristicas_json`) a la tabla `Products` para almacenar atributos dinámicos
   - Ejemplo de datos almacenados: categoría, subcategoría, especificaciones (tamaño, material, ingredientes, etc.)
   - Consultas SQL para extraer información del campo JSON

2. **Descuentos por volumen**:
   - Nueva tabla `volume_discounts` que define descuentos basados en cantidades mínimas compradas de un producto
   - Vista `analisis_volumen_descuento` que muestra análisis detallados de los descuentos

3. **Sistema de Categorías Jerárquicas**: Subcategorías para mejor organización de productos

## 🛠️ Tecnologías

- **PostgreSQL** 12+ 
- **pgAdmin** (opcional, para gestión gráfica)
- **Cliente psql** (para línea de comandos)

## 🚀 Instalación Rápida

### Prerrequisitos
- PostgreSQL 12 o superior instalado
- Acceso a la línea de comandos (para usar `psql`) o a pgAdmin

### Pasos de instalación:

1. **Clonar el repositorio**:
   ```bash
   git clone https://github.com/tu-usuario/northwind-postgres-modificado.git
   cd northwind-postgres-modificado
   ```

2. **Crear la base de datos**:
   ```bash
   createdb northwind_curso
   ```

3. **Restaurar el dump**:
   ```bash
   psql -d northwind_curso -f northwind_modificado.sql
   ```

## 🔍 Validar Instalación

Después de restaurar, ejecuta estas consultas para verificar:

```sql
-- Verificar la columna JSON en Products
SELECT product_id, product_name, caracteristicas_json 
FROM Products 
WHERE caracteristicas_json IS NOT NULL;

-- Verificar la tabla de descuentos por volumen
SELECT * FROM volume_discounts;

-- Probar la vista de análisis de descuentos
SELECT * FROM analisis_volumen_descuento;
```

## 📊 Nuevas Características Técnicas

### 1. Atributos dinámicos en productos (JSON)

**Modificación de la tabla Products**:
```sql
ALTER TABLE Products
ADD COLUMN caracteristicas_json JSON;
```

**Ejemplo de actualización de datos JSON**:
```sql
UPDATE Products SET caracteristicas_json = '{
  "categoria": "Bebidas",
  "subcategoria": "Calientes",
  "especificaciones": {
    "tamano": "500 ml",
    "material": "Vidrio",
    "ingredientes": ["Té", "Especias"]
  }
}' WHERE product_id = 1;
```

**Consultas sobre datos JSON**:
```sql
SELECT 
  caracteristicas_json->>'categoria' AS Categoria,
  caracteristicas_json->>'subcategoria' AS Subcategoria,
  caracteristicas_json->'especificaciones'->>'tamano' AS Tamaño
FROM Products
WHERE caracteristicas_json IS NOT NULL;
```

### 2. Descuentos por volumen

**Creación de tabla**:
```sql
CREATE TABLE volume_discounts (
  discount_id INT PRIMARY KEY,
  product_id INT NOT NULL,
  MinQuantity INT NOT NULL,
  Discount DECIMAL(4,2) NOT NULL CHECK (Discount BETWEEN 0 AND 1),
  FOREIGN KEY (product_id) REFERENCES Products(product_id)
);
```

**Vista de análisis**:
```sql
CREATE VIEW analisis_volumen_descuento AS
SELECT
  vd.discount_id,
  vd.product_id,
  p.product_name,
  p.unit_price,
  vd.MinQuantity,
  vd.Discount AS DiscountRate,
  ROUND((p.unit_price * (1 - vd.Discount))::NUMERIC, 2) AS DiscountedPrice,
  ROUND((p.unit_price * vd.Discount)::NUMERIC, 2) AS SavingsPerUnit,
  (vd.MinQuantity * p.unit_price) AS OriginalBatchPrice,
  ROUND((vd.MinQuantity * p.unit_price * (1 - vd.Discount))::NUMERIC, 2) AS DiscountedBatchPrice,
  ROUND((vd.MinQuantity * p.unit_price * vd.Discount)::NUMERIC, 2) AS TotalSavings
FROM volume_discounts vd
JOIN Products p ON vd.product_id = p.product_id;
```

## 📞 Soporte

Si tienes problemas con la instalación:
1. Verifica que PostgreSQL esté corriendo (`sudo service postgresql status`)
2. Asegúrate de tener permisos para crear bases de datos
3. Revisa que el archivo `northwind_modificado.sql` esté completo
4. Consulta los logs de PostgreSQL para errores detallados

---

**Nota**: Esta versión modificada de Northwind demuestra implementaciones avanzadas de PostgreSQL como campos JSON, vistas complejas y estructuras de datos jerárquicas.
