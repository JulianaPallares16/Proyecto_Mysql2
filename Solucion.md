# Ejercicios

## 1. Consultas SQL Especializadas

1. Como analista, quiero listar todos los productos con su empresa asociada y el precio más bajo por ciudad.
```sql
SELECT 
p.name AS producto,
c.name AS empresa,
cities.name AS ciudad,
cp.price AS precios
FROM products AS p
JOIN companyproducts AS cp ON p.id = cp.product_id
JOIN companies AS c ON cp.company_id = c.id
JOIN citiesormunicipalities AS cities ON c.city_id = cities.code
GROUP BY p.id, c.id, cities.name
ORDER BY cities.name, precios;
```

2. Como administrador, deseo obtener el top 5 de clientes que más productos han calificado en los últimos 6 meses.
```sql
SELECT 
c.name AS cliente,
COUNT(qp.rating) AS total_calificaciones
FROM quality_products AS qp
JOIN customers AS c ON c.id = qp.customer_id
WHERE qp.daterating >= CURDATE() - INTERVAL 6 MONTH
GROUP BY c.id, c.name
LIMIT 5;
```

3. Como gerente de ventas, quiero ver la distribución de productos por categoría y unidad de medida.
```sql
SELECT
p.name AS producto,
c.description AS categoria,
u.description AS unidad_medida
FROM products AS p
JOIN categories AS c ON c.id = p.category_id
JOIN companyproducts AS cp ON cp.product_id = p.id
JOIN unitofmeasure AS u ON u.id = cp.unitimeasure_id;
```

4. Como cliente, quiero saber qué productos tienen calificaciones superiores al promedio general.
```sql
SELECT DISTINCT
p.name AS producto
FROM quality_products AS qp
JOIN products AS p ON p.id = qp.product_id
WHERE qp.rating > (SELECT AVG(rating) FROM quality_products);

```

5. Como auditor, quiero conocer todas las empresas que no han recibido ninguna calificación.
```sql
SELECT 
c.id,
c.name AS empresa
FROM companies AS c
LEFT JOIN quality_products AS qp ON qp.company_id = c.id
WHERE qp.company_id IS NULL;

```

6. Como operador, deseo obtener los productos que han sido añadidos como favoritos por más de 10 clientes distintos.
```sql
SELECT
p.name AS producto,
COUNT(DISTINCT f.customer_id) AS clientes_favoritos
FROM products AS p
JOIN detail_favorites AS df ON df.product_id = p.id
JOIN favorites AS f ON f.id = df.favorite_id
GROUP BY p.name
HAVING clientes_favoritos > 10;

```

7. Como gerente regional, quiero obtener todas las empresas activas por ciudad y categoría.
```sql
SELECT 
c.name AS empresa,
ci.name AS ciudad_municipio,
ca.description AS categoria
FROM companies AS c
JOIN citiesormunicipalities AS ci ON ci.code = c.city_id
JOIN categories AS ca ON ca.id = c.category_id;
```

8. Como especialista en marketing, deseo obtener los 10 productos más calificados en cada ciudad.
```sql
SELECT 
ciudad,
producto,
total_calificaciones
FROM (
  SELECT 
  ci.name AS ciudad,
  p.name AS producto,
  COUNT(*) AS total_calificaciones,
  ROW_NUMBER() OVER (PARTITION BY ci.code ORDER BY COUNT(*) DESC) AS pos
  FROM quality_products AS qp
  JOIN customers AS cu ON cu.id = qp.customer_id
  JOIN citiesormunicipalities AS ci ON ci.code = cu.city_id
  JOIN products AS p ON p.id = qp.product_id
  GROUP BY ci.code, ci.name, p.id, p.name
) AS ranking
WHERE pos <= 10
ORDER BY ciudad, pos;
```

9. Como técnico, quiero identificar productos sin unidad de medida asignada.
```sql
SELECT 
p.name AS producto
FROM products AS p
LEFT JOIN companyproducts AS cp ON cp.product_id = p.id
LEFT JOIN unitofmeasure AS u ON u.id = cp.unitimeasure_id
WHERE cp.unitimeasure_id IS NULL;
```

10. Como gestor de beneficios, deseo ver los planes de membresía sin beneficios registrados.
```sql
SELECT 
m.name AS plan
FROM membershipsperiods AS mp
JOIN memberships AS m ON m.id = mp.membership_id
JOIN periods AS p ON p.id = mp.period_id
LEFT JOIN membershipsbenefits AS mb ON mb.membership_id = mp.membership_id AND mb.period_id = mp.period_id
WHERE mb.benefit_id IS NULL;
```

11. Como supervisor, quiero obtener los productos de una categoría específica con su promedio de calificación.
```sql
SELECT
c.description AS categoria,
p.name AS producto,
AVG(q.rating) AS promedio_calificación
FROM products AS p 
LEFT JOIN quality_products AS q ON q.product_id = p.id
JOIN categories AS c ON c.id = p.category_id
WHERE c.id = 5
GROUP BY c.description, p.name;
```

12. Como asesor, deseo obtener los clientes que han comprado productos de más de una empresa.
```sql
SELECT
c.id,
c.name AS cliente,
COUNT(DISTINCT qp.company_id) AS total_empresas
FROM quality_products AS qp
JOIN customers AS c ON c.id = qp.customer_id
GROUP BY c.id, c.name
HAVING total_empresas > 1;
```

13. Como director, quiero identificar las ciudades con más clientes activos.
```sql
SELECT
c.name AS ciudad_municipio,
COUNT(cl.city_id) AS clientes_activos
FROM citiesormunicipalities AS c
JOIN customers AS cl ON cl.city_id = c.code
GROUP BY c.name
ORDER BY clientes_activos DESC
LIMIT 5;
```

14. Como analista de calidad, deseo obtener el ranking de productos por empresa basado en la media de quality_products.
```sql
SELECT
c.name AS empresa,
p.name AS producto,
ROUND(AVG(qp.rating), 2) AS promedio_calificacion
FROM quality_products AS qp
JOIN products AS p ON p.id = qp.product_id
JOIN companies AS c ON c.id = qp.company_id
GROUP BY c.id, c.name, p.id, p.name
ORDER BY c.name, promedio_calificacion DESC;
```

15. Como administrador, quiero listar empresas que ofrecen más de cinco productos distintos.
```sql
SELECT
c.name AS empresa,
COUNT(DISTINCT cp.product_id) AS productos_ofrecidos
FROM companies AS c
JOIN companyproducts AS cp ON cp.company_id = c.id
GROUP BY c.name
HAVING productos_ofrecidos > 5;
```

16. Como cliente, deseo visualizar los productos favoritos que aún no han sido calificados.
```sql
SELECT
p.name AS producto_favorito_sin_calificar
FROM favorites AS f
JOIN detail_favorites AS df ON df.favorite_id = f.id
JOIN products AS p ON p.id = df.product_id
LEFT JOIN quality_products AS qp ON qp.product_id = p.id AND qp.customer_id = f.customer_id
WHERE f.customer_id = 1  AND qp.product_id IS NULL; 
```

17. Como desarrollador, deseo consultar los beneficios asignados a cada audiencia junto con su descripción.
```sql
SELECT 
a.description AS audiencia,
b.description AS beneficio,
b.detail AS descripcion
FROM audiences AS a
JOIN audiencebenefits AS ab ON ab.audience_id =  a.id
JOIN benefits AS b ON b.id = ab.benefit_id;
```

18. Como operador logístico, quiero saber en qué ciudades hay empresas sin productos asociados.
```sql
SELECT DISTINCT
ci.name AS ciudad,
c.name AS empresa
FROM companies AS c
JOIN citiesormunicipalities AS ci ON ci.code = c.city_id
LEFT JOIN companyproducts AS cp ON cp.company_id = c.id
WHERE cp.company_id IS NULL;
```

19. Como técnico, deseo obtener todas las empresas con productos duplicados por nombre.
```sql
SELECT
c.name AS empresa,
p.name AS producto,
COUNT(p.name) AS cantidad_duplicados
FROM companyproducts AS cp
JOIN companies AS c ON c.id = cp.company_id
JOIN products AS p ON p.id = cp.product_id
GROUP BY c.id, c.name, p.name
HAVING COUNT(p.name) > 1
ORDER BY c.name, p.name;

```

20. Como analista, quiero una vista resumen de clientes, productos favoritos y promedio de calificación recibido.
```sql
SELECT
c.name AS cliente_nombre,
COUNT(DISTINCT df.product_id) AS productos_favoritos,
ROUND(AVG(qp.rating), 2) AS promedio_calificacion_recibida
FROM customers AS c
LEFT JOIN favorites AS f ON f.customer_id = c.id
LEFT JOIN detail_favorites AS df ON df.favorite_id = f.id
LEFT JOIN quality_products AS qp ON qp.customer_id = c.id
GROUP BY c.id, c.name;
```

## 2. Subconsultas

1. Como gerente, quiero ver los productos cuyo precio esté por encima del promedio de su categoría.
```sql
SELECT 
    p.name AS producto,
    p.price AS precio,
    cat.description AS categoria
FROM products p
JOIN categories cat ON p.category_id = cat.id
WHERE 
    p.price > (
        SELECT AVG(p2.price)
        FROM products AS p2
        WHERE p2.category_id = p.category_id
    );
```

2. Como administrador, deseo listar las empresas que tienen más productos que la media de empresas.
```sql
SELECT 
    c.name AS empresa,
    COUNT(cp.product_id) AS total_productos
FROM companies c
JOIN companyproducts cp ON c.id = cp.company_id
GROUP BY c.id
HAVING 
    COUNT(cp.product_id) > (
        SELECT AVG(productos_por_empresa) 
        FROM (
            SELECT COUNT(product_id) AS productos_por_empresa
            FROM companyproducts
            GROUP BY company_id
        ) AS subquery
    );
```

3. Como cliente, quiero ver mis productos favoritos que han sido calificados por otros clientes.
```sql
SELECT 
    p.name AS producto,
    p.detail AS product_detail
FROM products p
WHERE 
    p.id IN (
        SELECT df.product_id
        FROM detail_favorites df
        WHERE df.favorite_id IN (
            SELECT f.id
            FROM favorites f
            WHERE f.customer_id = 1
        )
    )
    AND EXISTS (
        SELECT 1
        FROM quality_products q
        WHERE q.product_id = p.id AND q.customer_id <> 2
    );
```

4. Como supervisor, deseo obtener los productos con el mayor número de veces añadidos como favoritos.
```sql
SELECT 
    p.name AS producto,
    (
        SELECT COUNT(product_id)
        FROM detail_favorites df
        WHERE df.product_id = p.id
    ) AS veces_favorito
FROM products p
WHERE 
    p.id IN (
        SELECT DISTINCT df.product_id
        FROM detail_favorites df
    )
ORDER BY veces_favorito DESC;
```
5. Como técnico, quiero listar los clientes cuyo correo no aparece en la tabla rates ni en quality_products.
```sql
SELECT 
    c.name AS cliente,
    c.email
FROM customers c
WHERE 
    c.id NOT IN (
        SELECT DISTINCT r.customer_id
        FROM rates r
    )
    AND c.id NOT IN (
        SELECT DISTINCT q.customer_id
        FROM quality_products q
    );
```

6. Como gestor de calidad, quiero obtener los productos con una calificación inferior al mínimo de su categoría.
```sql
SELECT 
    p.name AS producto,
    cat.description AS categoria,
    (
        SELECT AVG(qp.rating)
        FROM quality_products qp
        WHERE qp.product_id = p.id
    ) AS promedio_producto
FROM products p
JOIN categories cat ON p.category_id = cat.id
WHERE 
    (
        SELECT AVG(qp.rating)
        FROM quality_products qp
        WHERE qp.product_id = p.id
    ) < (
        SELECT MIN(promedio_categoria)
        FROM (
            SELECT AVG(qp2.rating) AS promedio_categoria
            FROM products p2
            JOIN quality_products qp2 ON qp2.product_id = p2.id
            WHERE p2.category_id = p.category_id
            GROUP BY p2.id
        ) AS subquery
    );

```

7. Como desarrollador, deseo listar las ciudades que no tienen clientes registrados.
```sql
SELECT c.name AS Ciudad
FROM citiesormunicipalities c
WHERE NOT EXISTS (
    SELECT cu.city_id
    FROM customers cu
    WHERE cu.city_id = c.code
);
```

8. Como administrador, quiero ver los productos que no han sido evaluados en ninguna encuesta.
```sql
SELECT 
    p.name AS producto,
    p.detail,
    p.price
FROM products p
WHERE 
    p.id NOT IN (
        SELECT DISTINCT qp.product_id
        FROM quality_products qp
    );
```

9. Como auditor, quiero listar los beneficios que no están asignados a ninguna audiencia.
```sql
SELECT 
    b.description AS beneficio,
    b.detail
FROM benefits b
WHERE 
    b.id NOT IN (
        SELECT DISTINCT ab.benefit_id
        FROM audiencebenefits ab
    );
```

10. Como cliente, deseo obtener mis productos favoritos que no están disponibles actualmente en ninguna empresa.
```sql
SELECT 
    p.name AS Producto_NO_disponible
FROM detail_favorites AS df
JOIN favorites f ON df.favorite_id = f.id
JOIN products p ON df.product_id = p.id
WHERE f.customer_id = 1
  AND p.id NOT IN (
      SELECT cp.product_id
      FROM companyproducts AS cp
      WHERE cp.is_available = TRUE
  );
```

11. Como director, deseo consultar los productos vendidos en empresas cuya ciudad tenga menos de tres empresas registradas.
```sql
SELECT DISTINCT 
    p.name AS producto, 
    c.name AS empresa, 
    ci.name AS ciudad
FROM companyproducts cp
JOIN companies c ON cp.company_id = c.id
JOIN citiesormunicipalities ci ON c.city_id = ci.code
JOIN products p ON cp.product_id = p.id
WHERE c.city_id IN (
    SELECT c.city_id
    FROM companies c
    GROUP BY c.city_id
    HAVING COUNT(c.id) < 3
);
```

12. Como analista, quiero ver los productos con calidad superior al promedio de todos los productos.
```sql
SELECT
    p.name AS producto,
    qp.rating AS calificacion
FROM products p
JOIN quality_products qp ON qp.product_id = p.id
WHERE qp.rating > (
        SELECT AVG(qp2.rating)
        FROM quality_products qp2
    );
```

13. Como gestor, quiero ver empresas que sólo venden productos de una única categoría.
```sql
SELECT 
    c.name AS empresa
FROM companies c
WHERE c.id IN (
    SELECT cp.company_id
    FROM companyproducts cp
    JOIN products p ON cp.product_id = p.id
    GROUP BY cp.company_id
    HAVING COUNT(DISTINCT p.category_id) = 1
);
```

14. Como gerente comercial, quiero consultar los productos con el mayor precio entre todas las empresas.
```sql
SELECT 
    p.name AS producto, 
    cp.price, 
    c.name AS empresa
FROM companyproducts cp
JOIN products p ON cp.product_id = p.id
JOIN companies c ON cp.company_id = c.id
WHERE cp.price = (
    SELECT MAX(price)
    FROM companyproducts
);
```

15. Como cliente, quiero saber si algún producto de mis favoritos ha sido calificado por otro cliente con más de 4 estrellas.
```sql
SELECT DISTINCT 
    p.name AS producto
FROM detail_favorites df
JOIN favorites f ON df.favorite_id = f.id
JOIN products p ON df.product_id = p.id
WHERE f.customer_id = 3
    AND p.id IN (
        SELECT qp.product_id
        FROM quality_products qp
        WHERE qp.customer_id != 3 AND qp.rating > 4
  );
```

16. Como operador, quiero saber qué productos no tienen imagen asignada pero sí han sido calificados.
```sql
SELECT 
    p.name AS producto
FROM products p
WHERE (p.image IS NULL)
    AND p.id IN (
        SELECT DISTINCT qp.product_id
        FROM quality_products qp
  );
```

17. Como auditor, quiero ver los planes de membresía sin periodo vigente.
```sql
SELECT 
    m.name AS membresia
FROM memberships AS m
WHERE m.id NOT IN (
    SELECT DISTINCT mp.membership_id
    FROM membershipsperiods AS mp
);
```

18. Como especialista, quiero identificar los beneficios compartidos por más de una audiencia.
```sql
SELECT 
b.description AS beneficio
FROM benefits b
WHERE b.id IN (
    SELECT ab.benefit_id
    FROM audiencebenefits ab
    GROUP BY ab.benefit_id
    HAVING COUNT(DISTINCT ab.audience_id) > 1
);
```

19. Como técnico, quiero encontrar empresas cuyos productos no tengan unidad de medida definida.
```sql
SELECT
    c.name AS empresa
FROM companies c
WHERE c.id IN (
    SELECT cp.company_id
    FROM companyproducts cp
    WHERE cp.unitimeasure_id IS NULL
);
```

20. Como gestor de campañas, deseo obtener los clientes con membresía activa y sin productos favoritos.
```sql
SELECT
    c.name AS cliente
FROM customers c
WHERE c.membership_active = TRUE
  AND c.id NOT IN (
      SELECT DISTINCT f.customer_id
      FROM favorites f
  );
```

## 3. Funciones Agregadas

1. Obtener el promedio de calificación por producto
```sql
SELECT 
    p.name AS product_name,
    ROUND(AVG(qp.rating), 2) AS average_rating,
    COUNT(qp.rating) AS rating_count
FROM products AS p
JOIN quality_products AS qp ON p.id = qp.product_id
GROUP BY p.id, p.name
ORDER BY average_rating DESC;
```

2. Contar cuántos productos ha calificado cada cliente
```sql
SELECT 
    c.name AS customer_name,
    COUNT(DISTINCT qp.product_id) AS products_rated_count
FROM customers c
LEFT JOIN quality_products AS qp ON c.id = qp.customer_id
GROUP BY c.id, c.name
ORDER BY products_rated_count DESC;
```

3. Sumar el total de beneficios asignados por audiencia
```sql
SELECT  a.description AS audience_type, COUNT(af.benefit_id) AS total_benefits_assigned
FROM audiences AS a
JOIN audiencebenefits AS af ON a.id = af.audience_id
GROUP BY a.id, a.description;
```

4. Calcular la media de productos por empresa
```sql
SELECT AVG(e.c) AS promedio_productos_por_empresa
FROM (
    SELECT 
        cp.company_id AS i,
        COUNT(cp.product_id) AS c
    FROM companyproducts AS cp
    GROUP BY cp.company_id
) AS e;
```
5. Contar el total de empresas por ciudad
```sql
SELECT c.city_id,
    COUNT(*) AS total_empresas
FROM companies AS c
GROUP BY c.city_id;
```

6. Calcular el promedio de precios por unidad de medida
```sql
SELECT u.description AS unidad_medida, AVG(cp.price) AS promedio_precio
FROM companyproducts AS cp
JOIN unitofmeasure u ON cp. unitimeasure_id = u.id
GROUP BY u.description;
```

7. Contar cuántos clientes hay por ciudad
```sql
SELECT ci.name AS nombre_ciudad, COUNT(*) AS customer_city
FROM customers AS c
JOIN  citiesormunicipalities AS ci ON c.city_id = ci.code
GROUP BY ci.name;
```

8. Calcular planes de membresía por periodo
```sql
SELECT cm.start_date AS inicio,cm.end_date AS fin, COUNT(*) AS total_planes
FROM customers_memberships AS cm
GROUP BY cm.start_date, cm.end_date;
```

9. Ver el promedio de calificaciones dadas por un cliente a sus favoritos
```sql
SELECT f.customer_id AS cliente, AVG(qp.rating) AS promedio_calificacion
FROM favorites AS f
JOIN quality_products AS qp ON f.customer_id = qp.customer_id
JOIN detail_favorites AS df ON f.id = df.favorite_id AND df.product_id = qp.product_id
GROUP BY f.customer_id;
```

10. Consultar la fecha más reciente en que se calificó un producto
```sql
SELECT qp.product_id AS producto, MAX(qp.daterating) AS ultima_calificacion
FROM quality_products AS qp
GROUP BY qp.product_id;
```

11. Obtener la desviación estándar de precios por categoría
```sql
SELECT p.category_id AS categoria, STDDEV(cp.price) AS desviacion_precio
FROM companyproducts AS cp
JOIN products AS p ON cp.product_id = p.id
GROUP BY p.category_id;
```

12. Contar cuántas veces un producto fue favorito
```sql
SELECT p.name AS nombre_producto, COUNT(*) AS veces_favorito
FROM detail_favorites AS df
JOIN products AS p ON df.product_id = p.id
GROUP BY p.name;
```

13. Calcular el porcentaje de productos evaluados
```sql
SELECT 
    ROUND(
        (COUNT(DISTINCT qp.product_id) * 100.0 / 
        (SELECT COUNT(*) FROM products)), 
        2
    ) AS porcentaje_evaluados
FROM 
    quality_products qp;
```

14. Ver el promedio de rating por encuesta
```sql
SELECT p.name AS encuesta_name , AVG(r.rating) AS promedio_rating
FROM rates AS r
JOIN polls AS p ON r.poll_id = p.id
GROUP BY p.name;
```

15. Calcular el promedio y total de beneficios por plan
```sql
SELECT mb.membership_id, COUNT(*) AS total_beneficios, ROUND(AVG(b.id), 2) AS promedio_beneficios_id
FROM membershipsbenefits AS mb
JOIN benefits AS b ON mb.benefit_id = b.id
GROUP BY mb.membership_id;
```

16. Obtener media y varianza de precios por empresa
```sql
SELECT company_id,
    ROUND(AVG(price), 2) AS media_precio,
    ROUND(VARIANCE(price), 2) AS varianza_precio
FROM companyproducts 
GROUP BY company_id;
```

17. Ver total de productos disponibles en la ciudad del cliente
```sql
SELECT ciom.name AS ciudad, COUNT(DISTINCT cp.product_id) AS total_productos
FROM companyproducts AS cp
JOIN companies AS c ON cp.company_id = c.id
JOIN citiesormunicipalities AS ciom ON c.city_id = ciom.code
GROUP BY ciom.name;
```

18. Contar productos únicos por tipo de empresa
```sql
SELECT c.type_id AS tipo_empresa,
    COUNT(DISTINCT cp.product_id) AS productos_unicos
FROM companyproducts AS cp
JOIN companies AS c ON cp.company_id = c.id
GROUP BY c.type_id
ORDER BY c.type_id;
```

19. Ver total de clientes sin correo electrónico registrado
```sql
SELECT COUNT(c.email) AS total_sin_email
FROM customers AS c
WHERE email IS NULL;
```

20. Empresa con más productos calificados
```sql
SELECT c.name AS empresa,
COUNT(DISTINCT qp.product_id) AS total_productos_calificados
FROM quality_products AS qp
JOIN companies AS c ON qp.company_id = c.id
GROUP BY c.id, c.name
ORDER BY total_productos_calificados DESC
LIMIT 1;
```

## 4. Procedimientos Almacenados

1.  Registrar una nueva calificación y actualizar el promedio
```sql
DELIMITER // 
CREATE PROCEDURE insertar_promedio(
    IN pproduct_id INT,
    IN customer_id INT,
    IN poll_id INT,
    IN company_id VARCHAR(20),
    IN calificacion DOUBLE,
    OUT promedio DOUBLE
)
BEGIN
    INSERT INTO quality_products (product_id, customer_id, poll_id, company_id, rating)
    VALUES (pproduct_id, customer_id, poll_id, company_id, calificacion);

    SELECT AVG(rating)
    INTO promedio
    FROM quality_products
    WHERE product_id = pproduct_id;

    UPDATE products
    SET average_rating = promedio
    WHERE id = pproduct_id;

END //

DELIMITER ;

CALL insertar_promedio(20, 16, 3, '123456', 4.7, @prom);
```
2. Insertar empresa y asociar productos por defecto
DELIMITER //
```sql
CREATE PROCEDURE insert_empresa_product(
    IN c_id VARCHAR(20),
    IN type_id INT,
    IN nombre VARCHAR(50),
    IN category_id INT,
    IN city_id VARCHAR(6),
    IN audience_id INT,
    IN cel VARCHAR(15),
    IN email VARCHAR(80)
)
BEGIN
    DECLARE default_product_id INT;

    SELECT id INTO default_product_id
    FROM products
    ORDER BY id
    LIMIT 1;

    INSERT INTO companies (id, type_id, name, category_id, city_id, audience_id, cellphone, email)
    VALUES (c_id, type_id, nombre, category_id, city_id, audience_id, cel, email);

    INSERT INTO companyproducts (company_id, product_id)
    VALUES (c_id, default_product_id);

END
//

DELIMITER ;

CALL insert_empresa_product('1223456', 5, 'Exito', 4, '20001', 6, '121098765432', 'exito@gmail.com');
```

3. Añadir producto favorito validando duplicados
```sql
DELIMITER //

CREATE PROCEDURE agregar_favorito(
    IN p_customer_id INT,
    IN p_product_id INT
)
BEGIN
    DECLARE v_favorite_id INT;

    IF NOT EXISTS (
        SELECT 1
        FROM favorites AS f
        JOIN detail_favorites AS df ON df.favorite_id = f.id
        WHERE f.customer_id = p_customer_id
          AND df.product_id = p_product_id
    ) THEN

        SELECT id INTO v_favorite_id FROM favorites WHERE customer_id = p_customer_id LIMIT 1;

        IF v_favorite_id IS NULL THEN
            INSERT INTO favorites (customer_id) VALUES (p_customer_id);
            SET v_favorite_id = LAST_INSERT_ID();
        END IF;

        INSERT INTO detail_favorites (favorite_id, product_id)
        VALUES (v_favorite_id, p_product_id);
    END IF;

END
//
DELIMITER ;

CALL agregar_favorito(16, 20);
```

4. Generar resumen mensual de calificaciones por empresa
```sql
DELIMITER //

CREATE PROCEDURE resumen_mensual(
    IN p_year INT,
    IN p_month INT
)
BEGIN
    SELECT
        company_id,
        ROUND(AVG(rating), 2) AS avg_rating
    FROM rates
    WHERE YEAR(daterating) = p_year AND MONTH(daterating) = p_month
    GROUP BY company_id
    ORDER BY avg_rating DESC;
END //

DELIMITER ;

CALL resumen_mensual(2023, 01 );
```

5. Calcular beneficios activos por membresía
```sql
DELIMITER $$

CREATE PROCEDURE beneficio_activos_membresia ()
BEGIN
    SELECT 
        mb.membership_id,
        mb.period_id,
        mb.benefit_id,
        b.description AS beneficio,
        mp.start_date,
        mp.end_date,
        b.is_active
    FROM 
        membershipsbenefits mb
        INNER JOIN membershipsperiods mp ON mb.membership_id = mp.membership_id AND mb.period_id = mp.period_id
        INNER JOIN benefits b ON mb.benefit_id = b.id
    WHERE 
        mp.start_date <= CURDATE()
        AND mp.end_date >= CURDATE()
        AND b.is_active = TRUE;
END$$

DELIMITER ;
```

6. Eliminar productos huérfanos
```sql
DELIMITER //

CREATE PROCEDURE eliminar_productos_huerfanos()
BEGIN
    DELETE FROM products
    WHERE id NOT IN (
        SELECT DISTINCT product_id FROM quality_products
    )
    AND id NOT IN (
        SELECT DISTINCT product_id FROM companyproducts
    );
END //

DELIMITER ;

CALL eliminar_productos_huerfanos();
```

7. Actualizar precios de productos por categoría
```sql
DELIMITER //

CREATE PROCEDURE actualizar_precios_por_categoria (
    IN p_categoria_id INT,
    IN p_factor DECIMAL(5,2)
)
BEGIN
    UPDATE companyproducts AS cp
    JOIN products AS p ON cp.product_id = p.id
    SET cp.price = cp.price * p_factor
    WHERE p.category_id = p_categoria_id;
END //

DELIMITER ;

CALL actualizar_precios_por_categoria(3, 1.05);

```

8. Validar inconsistencia entre rates y quality_products
```sql
DELIMITER //
CREATE PROCEDURE inconsistencias_calificaciones()
BEGIN
    INSERT INTO errores_log (descripcion)
    SELECT
        CONCAT('Inconsistencia encontrada para customer_id=', r.customer_id,
               ', poll_id=', r.poll_id,
               ', company_id=', r.company_id)
    FROM rates r
    LEFT JOIN quality_products q
        ON r.customer_id = q.customer_id
        AND r.poll_id = q.poll_id
        AND r.company_id = q.company_id
    WHERE q.poll_id IS NULL;
END //
DELIMITER ;

CALL inconsistencias_calificaciones();
```

9. Asignar beneficios a nuevas audiencias
```sql
DELIMITER //

CREATE PROCEDURE asignar_beneficio_a_audiencia(
    IN p_benefit_id INT,
    IN p_audience_id INT
)
BEGIN
    IF NOT EXISTS (
        SELECT 1
        FROM audiencebenefits
        WHERE benefit_id = p_benefit_id
          AND audience_id = p_audience_id
    ) THEN

        INSERT INTO audiencebenefits (benefit_id, audience_id)
        VALUES (p_benefit_id, p_audience_id);
    END IF;
END //

DELIMITER ;

CALL asignar_beneficio_a_audiencia(3, 5);
```

10. Activar planes de membresía vencidos con pago confirmado
```sql
DELIMITER //

CREATE PROCEDURE activar_planes_vencidos()
BEGIN
    UPDATE customers_memberships
    SET status = 'ACTIVA'
    WHERE end_date < CURDATE()
      AND payment_confirmed = TRUE
      AND status <> 'ACTIVA';
END //

DELIMITER ;

CALL activar_planes_vencidos();
```

11. Listar productos favoritos del cliente con su calificación
```sql
DELIMITER //

CREATE PROCEDURE productos_favoritos_con_rating(
    IN p_customer_id INT
)
BEGIN
    SELECT 
        p.name AS producto,
        ROUND(AVG(r.rating), 2) AS promedio_calificacion
    FROM favorites f
    JOIN detail_favorites df ON df.favorite_id = f.id
    JOIN products p ON p.id = df.product_id
    LEFT JOIN quality_products r ON r.product_id = p.id
    WHERE f.customer_id = p_customer_id
    GROUP BY p.id, p.name;
END //

DELIMITER ;

CALL productos_favoritos_con_rating(1);
```

12. Registrar encuesta y sus preguntas asociadas
```sql
DELIMITER //

CREATE PROCEDURE registrar_encuesta_con_preguntas(
    IN p_name VARCHAR(80),
    IN p_description TEXT,
    IN p_isactive BOOLEAN,
    IN p_categorypoll_id INT,
    IN p_question1 TEXT,
    IN p_question2 TEXT,
    IN p_question3 TEXT
)
BEGIN
    DECLARE v_poll_id INT;

    INSERT INTO polls (name, description, isactive, categorypoll_id)
    VALUES (p_name, p_description, p_isactive, p_categorypoll_id);

    SET v_poll_id = LAST_INSERT_ID();

    INSERT INTO poll_questions (poll_id, question_text) VALUES
        (v_poll_id, p_question1),
        (v_poll_id, p_question2),
        (v_poll_id, p_question3);
END //

DELIMITER ;

CALL registrar_encuesta_con_preguntas(
    'Encuesta Julio Cliente',
    'Encuesta para conocer la satisfacción de clientes en julio',
    TRUE,
    1,
    '¿Cómo calificarías nuestro servicio?',
    '¿Qué mejorarías en la atención?',
    '¿Volverías a comprar con nosotros?'
);
```

13. Eliminar favoritos antiguos sin calificaciones
```sql
DELIMITER //

CREATE PROCEDURE eliminar_favoritos_antiguos_sin_calificacion()
BEGIN
    DELETE df
    FROM detail_favorites df
    JOIN favorites f ON df.favorite_id = f.id
    LEFT JOIN quality_products qp
        ON qp.product_id = df.product_id
        AND qp.customer_id = f.customer_id
    WHERE qp.product_id IS NULL
      AND df.created_at < DATE_SUB(CURDATE(), INTERVAL 12 MONTH);
END
//

DELIMITER ;
CALL eliminar_favoritos_antiguos_sin_calificacion();
```

14. Asociar beneficios automáticamente por audiencia
```sql
DELIMITER //

CREATE PROCEDURE asociar_beneficios_por_audiencia(
    IN p_audience_id INT
)
BEGIN
    INSERT INTO audiencebenefits (audience_id, benefit_id)
    SELECT p_audience_id, b.id
    FROM benefits b
    LEFT JOIN audiencebenefits ab ON ab.benefit_id = b.id AND ab.audience_id = p_audience_id
    WHERE ab.benefit_id IS NULL;
END //

DELIMITER ;

CALL asociar_beneficios_por_audiencia(3);

```

15. Historial de cambios de precio
```sql
DELIMITER //

CREATE PROCEDURE actualizar_precio_y_historial(
    IN p_company_id VARCHAR(20),
    IN p_product_id INT,
    IN p_nuevo_precio DOUBLE
)
BEGIN
    DECLARE v_precio_actual DOUBLE;

    SELECT price INTO v_precio_actual
    FROM companyproducts
    WHERE company_id = p_company_id AND product_id = p_product_id
    LIMIT 1;

    IF v_precio_actual IS NOT NULL AND v_precio_actual <> p_nuevo_precio THEN
        INSERT INTO historial_precios (company_id, product_id, old_price, new_price)
        VALUES (p_company_id, p_product_id, v_precio_actual, p_nuevo_precio);

        UPDATE companyproducts
        SET price = p_nuevo_precio
        WHERE company_id = p_company_id AND product_id = p_product_id;
    END IF;
END //

DELIMITER ;

CALL actualizar_precio_y_historial('123456', 10, 99.99);
```

16. Registrar encuesta activa automáticamente
```sql
DELIMITER //

CREATE PROCEDURE registrar_encuesta_activa(
    IN p_name VARCHAR(80),
    IN p_description TEXT,
    IN p_categorypoll_id INT
)
BEGIN
    INSERT INTO polls (name, description, isactive, categorypoll_id, start_date)
    VALUES (p_name, p_description, TRUE, p_categorypoll_id, NOW());
END //

DELIMITER ;

CALL registrar_encuesta_activa('Encuesta de aprobación', 'Encuesta para medir satisfacción del cliente', 2);
```

17. Actualizar unidad de medida de productos sin afectar ventas
```sql
DELIMITER //

CREATE PROCEDURE actualizar_unidad_medida(
    IN p_product_id INT,
    IN p_new_unit_id INT,
    OUT p_result VARCHAR(100)
)
BEGIN
    DECLARE ventas_existentes INT;

    SELECT COUNT(*) INTO ventas_existentes
    FROM quality_products
    WHERE product_id = p_product_id;

    IF ventas_existentes > 0 THEN
        SET p_result = 'No se puede actualizar: producto con ventas registradas.';
    ELSE
        UPDATE products
        SET unitimeasure_id = p_new_unit_id
        WHERE id = p_product_id;

        SET p_result = 'Unidad de medida actualizada correctamente.';
    END IF;
END //

DELIMITER ;

CALL actualizar_unidad_medida(10, 3, @resultado);
SELECT @resultado;

```

18. Recalcular promedios de calidad semanalmente
```sql
DELIMITER //

CREATE PROCEDURE recalcular_promedios_calidad()
BEGIN
    UPDATE products p
    JOIN (
        SELECT
            product_id,
            ROUND(AVG(rating), 2) AS avg_rating
        FROM quality_products
        GROUP BY product_id
    ) q ON p.id = q.product_id
    SET p.average_rating = q.avg_rating;
END //

DELIMITER ;

CALL recalcular_promedios_calidad();

```

19. Validar claves foráneas entre calificaciones y encuestas
```sql
DELIMITER //
DELIMITER //

CREATE PROCEDURE validar_claves_foraneas_polls()
BEGIN
    SELECT 
        r.poll_id,
        r.customer_id,
        r.company_id
    FROM rates r
    LEFT JOIN polls p ON r.poll_id = p.id
    WHERE r.poll_id IS NOT NULL
      AND p.id IS NULL;
END //

DELIMITER ;

CALL validar_claves_foraneas_polls();
```

20. Generar el top 10 de productos más calificados por ciudad
```sql
DELIMITER //

CREATE PROCEDURE top10_productos_por_ciudad()
BEGIN
    SELECT
        ci.name AS ciudad,
        p.name AS producto,
        COUNT(qp.rating) AS total_calificaciones
    FROM quality_products qp
    JOIN companies c ON c.id = qp.company_id
    JOIN citiesormunicipalities ci ON ci.code = c.city_id
    JOIN products p ON p.id = qp.product_id
    GROUP BY ci.code, ci.name, p.id, p.name
    ORDER BY ci.name, total_calificaciones DESC
    LIMIT 10;
END //

DELIMITER ;

CALL top10_productos_por_ciudad();
```

## 5. Triggers

1. Actualizar la fecha de modificación de un producto
```sql
DELIMITER //

CREATE TRIGGER trg_update_product_timestamp
BEFORE UPDATE ON products
FOR EACH ROW
BEGIN
    SET NEW.updated_at = NOW();
END;
//

DELIMITER ;
```

2. Registrar log cuando un cliente califica un producto
```sql
DELIMITER //

CREATE TRIGGER trg_log_rate_insert
AFTER INSERT ON rates
FOR EACH ROW
BEGIN
    INSERT INTO log_acciones (customer_id, company_id, poll_id, daterating, rating, accion)
    VALUES (NEW.customer_id, NEW.company_id, NEW.poll_id, NEW.daterating, NEW.rating, 'Cliente calificó un producto');
END
//

DELIMITER ;
```

3. Impedir insertar productos sin unidad de medida
```sql
DELIMITER //

CREATE TRIGGER trg_prevent_null_unitimeasure
BEFORE INSERT ON companyproducts
FOR EACH ROW
BEGIN
    IF NEW.unitimeasure_id IS NULL THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Error: unitimeasure_id no puede ser NULL.';
    END IF;
END;
//

DELIMITER ;
```

4. Validar calificaciones no mayores a 5
```sql
DELIMITER //

CREATE TRIGGER trg_validate_rating_before_insert
BEFORE INSERT ON rates
FOR EACH ROW
BEGIN
    IF NEW.rating > 5 THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Error: La calificación no puede ser mayor a 5.';
    END IF;
END;
//

DELIMITER ;
```

5. Actualizar estado de membresía cuando vence
```sql
DELIMITER //

CREATE TRIGGER trg_update_membership_status_before_update
BEFORE UPDATE ON customers_memberships
FOR EACH ROW
BEGIN
    IF NEW.end_date < CURDATE() THEN
        SET NEW.status = 'INACTIVA';
    ELSE
        SET NEW.status = 'ACTIVA';
    END IF;
END;
//

DELIMITER ;
```

6. Evitar duplicados de productos por empresa
```sql
DELIMITER //

CREATE TRIGGER trg_prevent_duplicate_product
BEFORE INSERT ON companyproducts
FOR EACH ROW
BEGIN
    IF EXISTS (
        SELECT 1 FROM companyproducts
        WHERE company_id = NEW.company_id
          AND product_id = NEW.product_id
    ) THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Error: Producto duplicado para esta empresa.';
    END IF;
END;
//

DELIMITER ;

```

7. Enviar notificación al añadir un favorito
```sql
DELIMITER //

CREATE TRIGGER trg_notify_favorite_added
AFTER INSERT ON detail_favorites
FOR EACH ROW
BEGIN
    DECLARE v_customer_id INT;

    SELECT customer_id INTO v_customer_id 
    FROM favorites 
    WHERE id = NEW.favorite_id;

    INSERT INTO notificaciones (customer_id, message, created_at)
    VALUES (v_customer_id, CONCAT('Producto con ID ', NEW.product_id, ' añadido a favoritos.'), NOW());
END;
//

DELIMITER ;

```

8. Insertar fila en quality_products tras calificación
```sql
DELIMITER //

CREATE TRIGGER trg_after_insert_rate
AFTER INSERT ON rates
FOR EACH ROW
BEGIN
    INSERT INTO quality_products (product_id, customer_id, poll_id, company_id, rating)
    VALUES (
        (SELECT cp.product_id FROM companyproducts cp WHERE cp.company_id = NEW.company_id LIMIT 1),
        NEW.customer_id,
        NEW.poll_id,
        NEW.company_id,
        NEW.rating
    );
END;
//

DELIMITER ;
```

9. Eliminar favoritos si se elimina el producto
```sql
DELIMITER //

CREATE TRIGGER trg_after_delete_product
AFTER DELETE ON products
FOR EACH ROW
BEGIN
    DELETE FROM detail_favorites
    WHERE product_id = OLD.id;
END;
//

DELIMITER ;
```

10. Bloquear modificación de audiencias activas
```sql
DELIMITER //

CREATE TRIGGER bloquear_modificacion_audiencia_en_uso
BEFORE UPDATE ON audiences
FOR EACH ROW
BEGIN
    IF EXISTS (
        SELECT 1 FROM audiencebenefits WHERE audience_id = OLD.id
    ) THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'No se puede modificar una audiencia que está en uso.';
    END IF;
END;
//

DELIMITER ;

UPDATE audiences SET description = 'Nombre Modificado' WHERE id = 1;
```

11. Recalcular promedio de calidad del producto tras nueva evaluación
```sql
DELIMITER //

CREATE TRIGGER actualizar_promedio_calidad
AFTER INSERT ON rates
FOR EACH ROW
BEGIN
    DECLARE nuevo_promedio DOUBLE;

    SELECT AVG(rating)
    INTO nuevo_promedio
    FROM quality_products
    WHERE product_id = NEW.product_id;

    UPDATE products
    SET average_rating = nuevo_promedio
    WHERE id = NEW.product_id;
END;
//

DELIMITER ;

INSERT INTO rates (customer_id, company_id, poll_id, daterating, rating)
VALUES (1, '123456', 1, NOW(), 4.5);
SELECT id, average_rating FROM products WHERE id = 1;
```

12. Registrar asignación de nuevo beneficio
```sql
DELIMITER //

CREATE TRIGGER trg_log_membershipbenefits_insert
AFTER INSERT ON membershipsbenefits
FOR EACH ROW
BEGIN
    INSERT INTO bitacora (tabla, mensaje)
    VALUES ('membershipsbenefits', 
            CONCAT('Nuevo beneficio asignado: membership_id=', NEW.membership_id,
                   ', period_id=', NEW.period_id,
                   ', audience_id=', NEW.audience_id,
                   ', benefit_id=', NEW.benefit_id));
END;
//

CREATE TRIGGER trg_log_audiencebenefits_insert
AFTER INSERT ON audiencebenefits
FOR EACH ROW
BEGIN
    INSERT INTO bitacora (tabla, mensaje)
    VALUES ('audiencebenefits',
            CONCAT('Nuevo beneficio asignado: audience_id=', NEW.audience_id,
                   ', benefit_id=', NEW.benefit_id));
END;
//

DELIMITER ;

INSERT INTO audiencebenefits (audience_id, benefit_id) VALUES (1, 2);
INSERT INTO membershipsbenefits (membership_id, period_id, audience_id, benefit_id)
VALUES (1, 1, 1, 2);
SELECT * FROM bitacora ORDER BY fecha DESC LIMIT 5;
```

13. Impedir doble calificación por parte del cliente
```sql
DELIMITER //

CREATE TRIGGER trg_no_double_rating_quality
BEFORE INSERT ON rates
FOR EACH ROW
BEGIN
    DECLARE last_rating DOUBLE;

    SELECT rating
    INTO last_rating
    FROM quality_products
    WHERE customer_id = NEW.customer_id
      AND poll_id = NEW.poll_id
      AND company_id = NEW.company_id
    ORDER BY 
        NULL 
    LIMIT 1;

    IF last_rating IS NOT NULL AND last_rating = NEW.rating THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'No puede calificar dos veces seguidas con la misma puntuación para este producto.';
    END IF;
END;
//

DELIMITER ;
```

14. Validar correos duplicados en clientes
```sql
DELIMITER //

CREATE TRIGGER trg_validar_email_unico
BEFORE INSERT ON customers
FOR EACH ROW
BEGIN
    IF EXISTS (
        SELECT 1 FROM customers WHERE email = NEW.email
    ) THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'El correo electrónico ya está registrado.';
    END IF;
END;
//

DELIMITER ;
INSERT INTO customers (name, city_id, audience_id, cellphone, email, address)
VALUES ('Juan Pérez', '05001', 4, '3101112222', 'juan.perez@email.com', 'Calle 123 #45-67');

```

15. Eliminar detalles de favoritos huérfanos
```sql
DELIMITER //

CREATE TRIGGER trg_eliminar_detalles_favoritos_huerfanos
AFTER DELETE ON favorites
FOR EACH ROW
BEGIN
    DELETE FROM detail_favorites
    WHERE favorite_id = OLD.id;
END;
//

DELIMITER ;
```

16. Actualizar campo updated_at en companies
```sql
DELIMITER //

CREATE TRIGGER trg_update_companies_updated_at
BEFORE UPDATE ON companies
FOR EACH ROW
BEGIN
    SET NEW.updated_at = NOW();
END;
//

DELIMITER ;
```

17. Impedir borrar ciudad si hay empresas activas
```sql
DELIMITER //

CREATE TRIGGER trg_no_delete_city_if_companies
BEFORE DELETE ON citiesormunicipalities
FOR EACH ROW
BEGIN
    IF EXISTS (
        SELECT 1
        FROM companies
        WHERE city_id = OLD.code
        LIMIT 1
    ) THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'No se puede eliminar la ciudad porque tiene empresas asociadas.';
    END IF;
END;
//

DELIMITER ;

DELETE FROM citiesormunicipalities WHERE code = '20001';
```

18. Registrar cambios de estado en encuestas
```sql
DELIMITER //

CREATE TRIGGER trg_log_status_change
AFTER UPDATE ON polls
FOR EACH ROW
BEGIN
  
    IF OLD.status <> NEW.status THEN
        INSERT INTO poll_status_log (poll_id, old_status, new_status, changed_at)
        VALUES (NEW.id, OLD.status, NEW.status, NOW());
    END IF;
END;
//

DELIMITER ;
```

19. Sincronizar rates y quality_products
```sql
DELIMITER //

CREATE TRIGGER trg_sync_quality_products
AFTER INSERT ON rates
FOR EACH ROW
BEGIN
    DECLARE v_exists INT;

    SELECT COUNT(*) INTO v_exists
    FROM quality_products
    WHERE customer_id = NEW.customer_id
      AND poll_id = NEW.poll_id
      AND company_id = NEW.company_id;

    IF v_exists > 0 THEN
        UPDATE quality_products
        SET rating = NEW.rating
        WHERE customer_id = NEW.customer_id
          AND poll_id = NEW.poll_id
          AND company_id = NEW.company_id;
    ELSE
        INSERT INTO quality_products (product_id, customer_id, poll_id, company_id, rating)
        VALUES (
            (SELECT product_id FROM companyproducts WHERE company_id = NEW.company_id LIMIT 1),
            NEW.customer_id,
            NEW.poll_id,
            NEW.company_id,
            NEW.rating
        );
    END IF;
END;
//

DELIMITER ;
```

20. Eliminar productos sin relación a empresas
```sql
DELIMITER //

CREATE TRIGGER trg_eliminar_producto_si_sin_empresa
AFTER DELETE ON companyproducts
FOR EACH ROW
BEGIN
    DECLARE v_count INT;

    SELECT COUNT(*) INTO v_count
    FROM companyproducts
    WHERE product_id = OLD.product_id;

    IF v_count = 0 THEN
        DELETE FROM products WHERE id = OLD.product_id;
    END IF;
END;
//

DELIMITER ;
```

## 6. Events

1. Borrar productos sin actividad cada 6 meses
```sql
DELIMITER //

CREATE EVENT eliminar_productos_sin_actividad
ON SCHEDULE EVERY 6 MONTH
DO
BEGIN
    DELETE FROM products
    WHERE id NOT IN (SELECT product_id FROM rates)
      AND id NOT IN (
          SELECT df.product_id
          FROM detail_favorites df
          JOIN favorites f ON df.favorite_id = f.id
      )
      AND id NOT IN (SELECT product_id FROM companyproducts);
END;
//

DELIMITER ;
```

2. Recalcular el promedio de calificaciones semanalmente
```sql
DELIMITER //

CREATE EVENT recalcular_promedios_semanal
ON SCHEDULE EVERY 1 WEEK
DO
BEGIN
    UPDATE product_metrics pm
    JOIN (
        SELECT product_id, AVG(rating) AS avg_rating
        FROM quality_products
        GROUP BY product_id
    ) q ON pm.product_id = q.product_id
    SET
        pm.average_rating = q.avg_rating,
        pm.updated_at = NOW();

    INSERT INTO product_metrics (product_id, average_rating, updated_at)
    SELECT product_id, AVG(rating), NOW()
    FROM quality_products
    WHERE product_id NOT IN (SELECT product_id FROM product_metrics)
    GROUP BY product_id;
END;
//

DELIMITER ;

```

3. Actualizar precios según inflación mensual
```sql
DELIMITER //

CREATE EVENT actualizar_precios_mensual
ON SCHEDULE EVERY 1 MONTH
DO
BEGIN
    UPDATE companyproducts
    SET price = price * 1.03;
END;
//

DELIMITER ;
```

4. Crear backups lógicos diariamente
```sql
DELIMITER //

CREATE EVENT backup_logico_diario
ON SCHEDULE EVERY 1 DAY
STARTS CURRENT_DATE + INTERVAL 1 DAY
DO
BEGIN
    DELETE FROM products_backup;
    DELETE FROM rates_backup;
    
    INSERT INTO products_backup SELECT * FROM products;
    INSERT INTO rates_backup SELECT * FROM rates;
END;
//

DELIMITER ;
```

5. Notificar sobre productos favoritos sin calificar
```sql
DELIMITER //

CREATE EVENT ev_recordar_favoritos_no_calificados
ON SCHEDULE EVERY 1 DAY
STARTS CURRENT_DATE + INTERVAL 1 DAY
DO
BEGIN
    DELETE FROM recordatorios;

    INSERT INTO recordatorios (customer_id, product_id, mensaje)
    SELECT f.customer_id, df.product_id,
           CONCAT('Recuerda calificar tu producto favorito con ID ', df.product_id)
    FROM favorites f
    JOIN detail_favorites df ON f.id = df.favorite_id
    LEFT JOIN quality_products qp
        ON qp.customer_id = f.customer_id AND qp.product_id = df.product_id
    WHERE qp.product_id IS NULL;
END;
//

DELIMITER ;

```

6. Revisar inconsistencias entre empresa y productos
```sql
DELIMITER //

CREATE EVENT ev_inconsistencias_empresa_producto
ON SCHEDULE
    EVERY 1 WEEK
    STARTS CURRENT_DATE + INTERVAL (7 - DAYOFWEEK(CURRENT_DATE)) DAY -- 
DO
BEGIN
    INSERT INTO errores_log (descripcion)
    SELECT CONCAT('Producto sin empresa: ID ', p.id)
    FROM products p
    WHERE NOT EXISTS (
        SELECT 1
        FROM companyproducts cp
        WHERE cp.product_id = p.id
    );

    INSERT INTO errores_log (descripcion)
    SELECT CONCAT('Empresa sin productos: ID ', c.id)
    FROM companies c
    WHERE NOT EXISTS (
        SELECT 1
        FROM companyproducts cp
        WHERE cp.company_id = c.id
    );
END;
//

DELIMITER ;

```

7. Archivar membresías vencidas diariamente
```sql
DELIMITER //

CREATE EVENT ev_archivar_membresias_vencidas
ON SCHEDULE
    EVERY 1 DAY
    STARTS CURRENT_DATE + INTERVAL 1 DAY
DO
BEGIN
    UPDATE customers_memberships
    SET status = 'INACTIVA'
    WHERE end_date < CURDATE()
      AND status <> 'INACTIVA';
END;
//

DELIMITER ;

```

8. Notificar beneficios nuevos a usuarios semanalmente
```sql
DELIMITER //

CREATE EVENT ev_notificar_beneficios_nuevos
ON SCHEDULE
    EVERY 1 WEEK
    STARTS CURRENT_DATE + INTERVAL 1 WEEK
DO
BEGIN
    INSERT INTO notificaciones (customer_id, message)
    SELECT 
        c.id AS customer_id,
        CONCAT('Nuevo beneficio disponible: ', b.description, ' - ', b.detail)
    FROM customers c
    JOIN benefits b
        ON b.created_at >= NOW() - INTERVAL 7 DAY;
END;
//

DELIMITER ;
```

9. Calcular cantidad de favoritos por cliente mensualmente
```sql
DELIMITER $$

CREATE PROCEDURE calcular_favoritos_mensual()
BEGIN
    INSERT INTO favoritos_resumen (customer_id, total_favoritos, mes_resumen)
    SELECT 
        f.customer_id,
        COUNT(df.product_id),
        CURDATE()
    FROM favorites f
    JOIN details_favorites df ON f.id = df.favorite_id
    GROUP BY f.customer_id;
END$$

DELIMITER ;


CREATE EVENT IF NOT EXISTS evt_calcular_favoritos_mensual
ON SCHEDULE EVERY 1 MONTH
STARTS CURRENT_DATE + INTERVAL 1 MONTH
DO
    CALL calcular_favoritos_mensual();
```

10. Validar claves foráneas semanalmente
```sql
DELIMITER $$

CREATE PROCEDURE validar_claves_foraneas()
BEGIN
    INSERT INTO inconsistencias_fk (tabla_afectada, descripcion)
    SELECT 
        'companyproducts',
        CONCAT('Producto con ID ', cp.product_id, ' no existe en products.')
    FROM companyproducts cp
    LEFT JOIN products p ON cp.product_id = p.id
    WHERE p.id IS NULL;

    INSERT INTO inconsistencias_fk (tabla_afectada, descripcion)
    SELECT 
        'favorites',
        CONCAT('Cliente con ID ', f.customer_id, ' no existe en customers.')
    FROM favorites f
    LEFT JOIN customers c ON f.customer_id = c.id
    WHERE c.id IS NULL;

    INSERT INTO inconsistencias_fk (tabla_afectada, descripcion)
    SELECT 
        'details_favorites',
        CONCAT('Producto con ID ', df.product_id, ' no existe en products.')
    FROM details_favorites df
    LEFT JOIN products p ON df.product_id = p.id
    WHERE p.id IS NULL;

    INSERT INTO inconsistencias_fk (tabla_afectada, descripcion)
    SELECT 
        'rates',
        CONCAT('Cliente con ID ', r.customer_id, ' no existe en customers.')
    FROM rates r
    LEFT JOIN customers c ON r.customer_id = c.id
    WHERE c.id IS NULL;
END$$

DELIMITER ;

CREATE EVENT IF NOT EXISTS evt_validar_claves_foraneas
ON SCHEDULE EVERY 1 WEEK
STARTS CURRENT_DATE + INTERVAL 1 WEEK
DO
    CALL validar_claves_foraneas();
```

11. Eliminar calificaciones inválidas antiguas
```sql
DELIMITER $$

CREATE PROCEDURE eliminar_calificaciones_invalidas()
BEGIN
    DELETE FROM rates
    WHERE (rating IS NULL OR rating < 0)
      AND created_at < NOW() - INTERVAL 3 MONTH;
END$$

DELIMITER ;

CREATE EVENT IF NOT EXISTS evt_eliminar_calificaciones_invalidas
ON SCHEDULE
    EVERY 1 MONTH
    STARTS CURRENT_DATE + INTERVAL 1 MONTH
DO
    CALL eliminar_calificaciones_invalidas();
```

12. Cambiar estado de encuestas inactivas automáticamente
```sql
DELIMITER $$

CREATE PROCEDURE inactivar_encuestas_antiguas()
BEGIN
    UPDATE polls
    SET isactive = FALSE
    WHERE isactive = TRUE
      AND id NOT IN (
          SELECT DISTINCT poll_id
          FROM rates
          WHERE daterating >= NOW() - INTERVAL 6 MONTH
      );
END$$

DELIMITER ;

CREATE EVENT IF NOT EXISTS evt_inactivar_encuestas_antiguas
ON SCHEDULE 
    EVERY 1 MONTH
    STARTS CURRENT_DATE + INTERVAL 1 MONTH
DO
    CALL inactivar_encuestas_antiguas();
```

13. Registrar auditorías de forma periódica
```sql
DELIMITER $$

CREATE PROCEDURE registrar_auditoria_diaria()
BEGIN
    INSERT INTO auditorias_diarias (
        fecha,
        total_productos,
        total_clientes,
        total_empresas,
        total_calificaciones
    )
    SELECT
        CURDATE(),
        (SELECT COUNT(id) FROM products),
        (SELECT COUNT(id) FROM customers),
        (SELECT COUNT(id) FROM companies),
        (SELECT COUNT(rating) FROM rates);
END$$

DELIMITER ;

CREATE EVENT IF NOT EXISTS evt_registrar_auditoria_diaria
ON SCHEDULE
    EVERY 1 DAY
    STARTS CURRENT_DATE + INTERVAL 1 DAY
DO
    CALL registrar_auditoria_diaria();
```

14. Notificar métricas de calidad a empresas
```sql
DELIMITER $$

CREATE PROCEDURE notificar_metricas_calidad()
BEGIN
    INSERT INTO notificaciones_empresa (
        company_id,
        product_id,
        promedio_calidad,
        fecha_envio
    )
    SELECT
        qp.company_id,
        qp.product_id,
        AVG(qp.rating) AS promedio_calidad,
        NOW()
    FROM quality_products qp
    GROUP BY qp.company_id, qp.product_id;
END$$

DELIMITER ;

CREATE EVENT IF NOT EXISTS evt_notificar_metricas_calidad
ON SCHEDULE
    EVERY 1 WEEK
    STARTS CURRENT_DATE + INTERVAL (7 - WEEKDAY(CURRENT_DATE)) DAY
DO
    CALL notificar_metricas_calidad();
```

15. Recordar renovación de membresías
```sql
DELIMITER $$

CREATE PROCEDURE recordar_renovacion_membresias()
BEGIN
    INSERT INTO recordatorios_membresia (
        membership_id,
        period_id,
        mensaje,
        fecha_recordatorio
    )
    SELECT
        mp.membership_id,
        mp.period_id,
        CONCAT('Tu membresía vencerá el ', DATE_FORMAT(mp.end_date, '%Y-%m-%d'), '. ¡Renuévala a tiempo!'),
        NOW()
    FROM membershipperiods mp
    WHERE mp.end_date BETWEEN CURDATE() AND CURDATE() + INTERVAL 7 DAY;
END$$

DELIMITER ;

CREATE EVENT IF NOT EXISTS evt_recordar_renovacion_membresias
ON SCHEDULE
    EVERY 1 DAY
    STARTS CURRENT_DATE + INTERVAL 1 DAY
DO
    CALL recordar_renovacion_membresias();
```

16. Reordenar estadísticas generales cada semana
```sql
DELIMITER $$

CREATE PROCEDURE actualizar_estadisticas()
BEGIN
    INSERT INTO estadisticas (
        fecha,
        total_productos,
        total_empresas,
        total_clientes,
        total_membresias_activas
    )
    SELECT
        NOW(),
        (SELECT COUNT(p.id) FROM products p),
        (SELECT COUNT(c.id) FROM companies c),
        (SELECT COUNT(cu.id) FROM customers cu),
        (SELECT COUNT(mp.membership_id) 
         FROM membershipperiods mp 
         WHERE mp.status = 'ACTIVA');
END$$

DELIMITER ;

CREATE EVENT IF NOT EXISTS evt_actualizar_estadisticas
ON SCHEDULE
    EVERY 1 WEEK
    STARTS CURRENT_DATE + INTERVAL 1 WEEK
DO
    CALL actualizar_estadisticas();
```

17. Crear resúmenes temporales de uso por categoría
```sql
DELIMITER $$

CREATE PROCEDURE generar_resumen_uso_por_categoria()
BEGIN
    INSERT INTO resumen_uso_categorias (
        categoria_id,
        nombre_categoria,
        cantidad_calificados,
        fecha
    )
    SELECT
        c.id,
        c.description,
        COUNT(qp.product_id),
        NOW()
    FROM quality_products qp
    JOIN products p ON qp.product_id = p.id
    JOIN categories c ON p.category_id = c.id
    GROUP BY c.id, c.description;
END$$

DELIMITER ;

CREATE EVENT IF NOT EXISTS evt_resumen_uso_por_categoria
ON SCHEDULE
    EVERY 1 WEEK
    STARTS CURRENT_DATE + INTERVAL 1 WEEK
DO
    CALL generar_resumen_uso_por_categoria();
```

18. Actualizar beneficios caducados
```sql
DELIMITER $$

CREATE PROCEDURE actualizar_beneficios_caducados()
BEGIN
    UPDATE benefits
    SET is_active = FALSE
    WHERE expires_at IS NOT NULL
      AND expires_at < CURDATE()
      AND is_active = TRUE;
END$$

DELIMITER ;

CREATE EVENT IF NOT EXISTS evt_actualizar_beneficios_caducados
ON SCHEDULE
    EVERY 1 DAY
    STARTS CURRENT_DATE + INTERVAL 1 DAY
DO
    CALL actualizar_beneficios_caducados();
```

19. Alertar productos sin evaluación anual
```sql
DELIMITER $$

CREATE PROCEDURE alertar_productos_sin_evaluacion_anual()
BEGIN
    INSERT INTO alertas_productos (product_id, mensaje, fecha_alerta)
    SELECT p.id,
           CONCAT('El producto "', p.name, '" no ha sido evaluado en el último año.') AS mensaje,
           NOW() AS fecha_alerta
    FROM products p
    WHERE NOT EXISTS (
        SELECT q.product_id
        FROM quality_products q
        WHERE q.product_id = p.id
          AND q.daterating >= CURDATE() - INTERVAL 365 DAY
    );
END$$

DELIMITER ;

CREATE EVENT IF NOT EXISTS evt_alertar_productos_sin_evaluacion_anual
ON SCHEDULE
    EVERY 1 DAY
    STARTS CURRENT_DATE + INTERVAL 1 DAY
DO
    CALL alertar_productos_sin_evaluacion_anual();
```

20. Actualizar precios con índice externo
```sql
DELIMITER $$

CREATE PROCEDURE actualizar_precios_con_indice()
BEGIN
    DECLARE indice DOUBLE;

    SELECT valor INTO indice
    FROM inflacion_indice
    ORDER BY fecha_aplicacion DESC
    LIMIT 1;

    UPDATE companyproducts
    SET price = price * indice;
END$$

DELIMITER ;

CREATE EVENT IF NOT EXISTS evt_actualizar_precios_con_indice
ON SCHEDULE
    EVERY 1 MONTH
    STARTS CURRENT_DATE + INTERVAL 1 MONTH
DO
    CALL actualizar_precios_con_indice();
```

## 7. Historias de Usuario con JOINs

1. Ver productos con la empresa que los vende
```sql
SELECT
c.name AS empresa,
p.name AS producto,
cp.price AS price
FROM companyproducts AS cp
INNER JOIN companies AS c ON c.id = cp.company_id
INNER JOIN products AS p ON p.id = cp.product_id;

```

2. Mostrar productos favoritos con su empresa y categoría
```sql
SELECT 
p.name AS productos_favoritos,
c.name AS empresa,
ca.description AS categoria
FROM detail_favorites AS df
JOIN products AS p ON p.id = df.product_id 
JOIN categories AS ca ON ca.id = p.category_id
JOIN favorites AS f ON f.id = df.favorite_id
JOIN companies AS c ON c.id = f.company_id;
```

3. Ver empresas aunque no tengan productos
```sql
SELECT
c.name AS empresa
FROM companies AS c
LEFT JOIN companyproducts AS cp ON cp.company_id = c.id;
```

4. Ver productos que fueron calificados (o no)
```sql
SELECT 
p.name AS producto,
r.rating
FROM quality_products AS  r
RIGHT JOIN products AS p ON p.id = r.product_id;
```

5. Ver productos con promedio de calificación y empresa
```sql
SELECT 
p.name AS producto,
c.name AS empresa,
AVG(r.rating) AS promedio_calificación
FROM products AS p
JOIN companyproducts AS cp ON cp.product_id = p.id
JOIN companies AS c ON c.id = cp.company_id
JOIN rates AS r ON r.company_id = c.id
GROUP BY p.name, c.name;
```

6. Ver clientes y sus calificaciones (si las tienen)
```sql
SELECT 
c.name AS cliente,
q.rating AS calificaciones 
FROM customers AS c
LEFT JOIN quality_products AS q ON q.customer_id = c.id;
```

7. Ver favoritos con la última calificación del cliente
```sql
SELECT 
p.name AS producto_favorito,
c.name AS empresa,
r.rating AS ultima_calificacion,
r.daterating AS fecha_calificacion
FROM favorites f
JOIN detail_favorites df ON df.favorite_id = f.id
JOIN products p ON p.id = df.product_id
JOIN companies c ON c.id = f.company_id
JOIN rates r ON r.company_id = f.company_id AND r.customer_id = f.customer_id
WHERE r.daterating = (
    SELECT MAX(r2.daterating)
    FROM rates r2
    WHERE r2.customer_id = f.customer_id AND r2.company_id = f.company_id
)
AND f.customer_id = 1;
```

8. Ver beneficios incluidos en cada plan de membresía
```sql
SELECT
m.name AS plan,
b.description AS beneficios
FROM membershipsbenefits AS mb
JOIN memberships AS m ON m.id = mb.membership_id
JOIN benefits AS b ON b.id = mb.benefit_id;
```

9. Ver clientes con membresía activa y sus beneficios
```sql
SELECT 
c.name AS customer_name,
m.name AS membership_name,
p.name AS period_name,
b.description AS benefit,
b.detail
FROM customers AS c
JOIN customers_memberships AS cm ON c.id = cm.customer_id
JOIN memberships AS m ON cm.membership_id = m.id
JOIN periods AS p ON cm.period_id = p.id
JOIN membershipsbenefits AS mb ON mb.membership_id = m.id AND mb.period_id = p.id AND mb.audience_id = c.audience_id
JOIN benefits b ON b.id = mb.benefit_id
WHERE CURRENT_DATE BETWEEN cm.start_date AND cm.end_date;
```

10. Ver ciudades con cantidad de empresas
```sql
SELECT
c.name AS ciudad_municipio,
COUNT(co.city_id) AS numero_empresas
FROM citiesormunicipalities AS c 
JOIN companies AS co ON co.city_id = c.code
GROUP BY c.name;
```

11. Ver encuestas con calificaciones
```sql
SELECT 
c.name AS customer_name,
co.name AS company_name,
r.poll_id,
p.name AS poll_name,
r.rating AS calificación
FROM rates AS r
JOIN polls AS p ON r.poll_id = p.id
JOIN customers AS c ON r.customer_id = c.id
JOIN companies AS co ON r.company_id = co.id;
```

12. Ver productos evaluados con datos del cliente
```sql
SELECT
p.name AS product_name,
c.name AS customer_name,
qp.daterating
FROM quality_products AS qp
JOIN customers AS c ON qp.customer_id = c.id
JOIN products AS p ON qp.product_id = p.id
JOIN companies AS comp ON qp.company_id = comp.id;
```

13. Ver productos con audiencia de la empresa
```sql
SELECT
p.name AS producto,
c.name AS empresa,
a.description AS audiencia
FROM products AS p 
JOIN companyproducts AS cp ON cp.product_id = p.id
JOIN companies AS c ON c.id = cp.company_id
JOIN audiences AS a ON a.id = c.audience_id;
```

14. Ver clientes con sus productos favoritos
```sql
SELECT 
c.name AS customer,
p.name AS productos_favoritos
FROM customers AS c 
JOIN favorites AS f ON f.customer_id = c.id
JOIN detail_favorites AS df ON df.favorite_id = f.id
JOIN products AS p ON p.id = df.product_id;
```

15. Ver planes, periodos, precios y beneficios
```sql
SELECT 
m.name AS membresía,
p.name AS periodo,
ms.price AS precio,
b.description AS beneficios
FROM memberships AS m
JOIN membershipsperiods AS ms ON ms.membership_id = m.id
JOIN periods AS p ON p.id = ms.period_id
JOIN membershipsbenefits AS mb ON mb.membership_id = ms.membership_id AND mb.period_id = ms.period_id 
JOIN benefits AS b ON b.id = mb. benefit_id
```

16. Ver combinaciones empresa-producto-cliente calificados
```sql
SELECT
c.name AS empresa,
p.name AS producto,
cl.name AS cliente,
q.rating AS calificación
FROM quality_products AS q
JOIN companies AS c ON c.id = q.company_id
JOIN products AS p ON p.id = q.product_id
JOIN customers AS cl ON cl.id = q.customer_id
JOIN companyproducts AS cp ON cp.product_id = q.product_id AND cp.company_id = q.company_id;
```

17. Comparar favoritos con productos calificados
```sql
SELECT 
p.name AS producto
FROM favorites AS f
JOIN detail_favorites AS df ON df.favorite_id = f.id
JOIN products AS p ON p.id = df.product_id
JOIN quality_products AS q ON q.product_id = df.product_id AND q.customer_id = f.customer_id
WHERE f.customer_id = 1;
```

18. Ver productos ordenados por categoría
```sql
SELECT 
p.name AS producto,
c.description AS categoria
FROM products AS p
JOIN categories AS c ON c.id = p.category_id;
```

19. Ver beneficios por audiencia, incluso vacíos
```sql
SELECT 
a.description AS audiencia,
b.description AS beneficio
FROM audiences AS a
LEFT JOIN audiencebenefits AS ab ON ab.audience_id = a.id
LEFT JOIN benefits AS b ON b.id = ab.benefit_id;

```

20. Ver datos cruzados entre calificaciones, encuestas, productos y clientes
```sql
SELECT 
c.name AS cliente,
p.name AS producto,
pl.name AS encuesta,
pl.description AS descripcion_encuesta,
qp.rating AS calificacion
FROM quality_products AS qp
JOIN customers AS c ON c.id = qp.customer_id
JOIN products AS p ON p.id = qp.product_id
JOIN polls AS pl ON pl.id = qp.poll_id;
```

## 8. Historias de Usuario con Funciones Definidas por el Usuario (UDF)

1. Como analista, quiero una función que calcule el promedio ponderado de calidad de un producto basado en sus calificaciones y fecha de evaluación.
```sql
DELIMITER //

CREATE FUNCTION calcular_promedio_ponderado(pid INT)
RETURNS DOUBLE
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE promedio DOUBLE;

    SELECT 
        SUM(rating * (1 / (1 + DATEDIFF(CURDATE(), DATE(daterating))))) / 
        SUM(1 / (1 + DATEDIFF(CURDATE(), DATE(daterating))))
    INTO promedio
    FROM quality_products
    WHERE product_id = pid;

    RETURN IFNULL(promedio, 0);
END //

DELIMITER ;

SELECT calcular_promedio_ponderado(5);
```

2. Como auditor, deseo una función que determine si un producto ha sido calificado recientemente (últimos 30 días).
```sql
DELIMITER //

CREATE FUNCTION es_calificacion_reciente(fecha DATETIME)
RETURNS BOOLEAN
DETERMINISTIC
BEGIN
    RETURN fecha >= CURDATE() - INTERVAL 30 DAY;
END //

DELIMITER ;

SELECT es_calificacion_reciente('2025-07-10'); 
SELECT es_calificacion_reciente('2025-06-01');
```

3. Como desarrollador, quiero una función que reciba un product_id y devuelva el nombre completo de la empresa que lo vende.
```sql
DELIMITER //

CREATE FUNCTION obtener_empresa_producto(producto_id INT)
RETURNS VARCHAR(80)
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE nombre_empresa VARCHAR(80);

    SELECT c.name
    INTO nombre_empresa
    FROM companies c
    JOIN companyproducts cp ON c.id = cp.company_id
    WHERE cp.product_id = producto_id
    LIMIT 1;

    RETURN nombre_empresa;
END //

DELIMITER ;


SELECT obtener_empresa_producto(9);
```

4. Como operador, deseo una función que, dado un customer_id, me indique si el cliente tiene una membresía activa.
```sql
DELIMITER //

CREATE FUNCTION tiene_membresia_activa(customer_id INT)
RETURNS BOOLEAN
DETERMINISTIC 
READS SQL DATA 
BEGIN 
    DECLARE existe INT;

    SELECT COUNT(*) INTO existe
    FROM customers_memberships 
    WHERE customer_id
        AND CURDATE() BETWEEN start_date AND end_date;

    RETURN existe > 0;
END //

DELIMITER ;


SELECT tiene_membresia_activa(1);

```

5. Como administrador, quiero una función que valide si una ciudad tiene más de X empresas registradas, recibiendo la ciudad y el número como parámetros.
```sql
DELIMITER //

CREATE FUNCTION ciudad_supera_empresas(cid VARCHAR(15), limite INT)
RETURNS BOOLEAN
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE total INT;

    SELECT COUNT(*) INTO total
    FROM companies
    WHERE city_id = cid;

    IF total > limite THEN
        RETURN TRUE;
    ELSE
        RETURN FALSE;
    END IF;
END  //

DELIMITER ;

SELECT ciudad_supera_empresas('11001', 1);
```

6. Como gerente, deseo una función que, dado un rate_id, me devuelva una descripción textual de la calificación (por ejemplo, “Muy bueno”, “Regular”).
```sql
DELIMITER //

CREATE FUNCTION descripcion_calificacion(valor DOUBLE)
RETURNS VARCHAR(20)
DETERMINISTIC
BEGIN
    DECLARE calificacion VARCHAR(20);

    IF valor >= 5 THEN 
        SET calificacion = 'Excelente';
    ELSEIF valor = 4 THEN
        SET calificacion = 'Bueno';
    ELSEIF valor = 3 THEN
        SET calificacion = 'Regular';
    ELSEIF valor = 2 THEN
        SET calificacion = 'Mala';
    ELSE 
        SET calificacion = 'Muy mala';
    END IF;

    RETURN calificacion;
END;
//

DELIMITER ;

SELECT descripcion_calificacion(5); 
SELECT descripcion_calificacion(4); 
SELECT descripcion_calificacion(2.5); 
SELECT descripcion_calificacion(1); 
```

7. Como técnico, quiero una función que devuelva el estado de un producto en función de su evaluación (ej. “Aceptable”, “Crítico”).
```sql
DELIMITER //

CREATE FUNCTION estado_producto(product_id INT)
RETURNS VARCHAR(15)
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE promedio DOUBLE;
    DECLARE estado VARCHAR(15);

    SELECT AVG(rating) INTO promedio
    FROM quality_products
    WHERE product_id ;

    IF promedio IS NULL THEN
        SET estado = 'Sin datos';
    ELSEIF promedio < 3 THEN
        SET estado = 'Crítico';
    ELSEIF promedio < 4 THEN
        SET estado = 'Aceptable';
    ELSE
        SET estado = 'Óptimo';
    END IF;

    RETURN estado;
END //

DELIMITER ;

SELECT estado_producto(5);
```

8. Como cliente, deseo una función que indique si un producto está entre mis favoritos, recibiendo el product_id y mi customer_id.
```sql
DELIMITER //

CREATE FUNCTION es_favorite(cid INT, pid INT)
RETURNS BOOLEAN
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE existe INT;

    SELECT COUNT(*) INTO existe
    FROM favorites AS f
    JOIN detail_favorites AS df ON f.id = df.favorite_id
    WHERE f.customer_id = cid
      AND df.product_id = pid;

    RETURN existe > 0;
END //

DELIMITER ;

SELECT es_favorite(3, 7);
```

9. Como gestor de beneficios, quiero una función que determine si un beneficio está asignado a una audiencia específica, retornando verdadero o falso.
```sql
DELIMITER //

CREATE FUNCTION beneficio_asignado_audiencia(benefit_id INT, audience_id INT)
RETURNS BOOLEAN
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE existe INT;

    SELECT COUNT(*) INTO existe
    FROM audiencebenefits
    WHERE benefit_id 
      AND audience_id ;

    RETURN existe > 0;
END //

DELIMITER ;

SELECT beneficio_asignado_audiencia(4, 2);
```

10. Como auditor, deseo una función que reciba una fecha y determine si se encuentra dentro de un rango de membresía activa.
```sql
DELIMITER //

CREATE FUNCTION fecha_en_membresia(fecha DATE, cid INT)
RETURNS BOOLEAN
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE existe INT;

    SELECT COUNT(*) INTO existe
    FROM customers_memberships
    WHERE customer_id = cid
      AND fecha BETWEEN start_date AND end_date;

    RETURN existe > 0;
END;
//

DELIMITER ;

SELECT fecha_en_membresia('2025-07-10', 3);
```

11. Como desarrollador, quiero una función que calcule el porcentaje de calificaciones positivas de un producto respecto al total.
```sql
DELIMITER //

CREATE FUNCTION porcentaje_positivas(product_id INT)
RETURNS DOUBLE
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE total INT DEFAULT 0;
    DECLARE positivas INT DEFAULT 0;
    DECLARE porcentaje DOUBLE;

    SELECT COUNT(*) INTO total
    FROM quality_products
    WHERE product_id;

    IF total = 0 THEN
        RETURN 0;
    END IF;

    SELECT COUNT(*) INTO positivas
    FROM quality_products
    WHERE product_id  AND rating >= 4;

    SET porcentaje = (positivas * 100.0) / total;
    RETURN porcentaje;
END //

DELIMITER ;

SELECT porcentaje_positivas(7);
```

12. Como supervisor, deseo una función que calcule la edad de una calificación, en días, desde la fecha actual.
```sql
DELIMITER //

CREATE FUNCTION edad_calificacion(rate_id INT)
RETURNS INT
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE fecha_calificacion DATETIME;
    DECLARE edad INT;

    SELECT daterating INTO fecha_calificacion
    FROM rates
    WHERE customer_id = rate_id;
    SET edad = DATEDIFF(CURRENT_DATE, fecha_calificacion);

    RETURN edad;
END //

DELIMITER ;

SELECT edad_calificacion(3);
```

13. Como operador, quiero una función que, dado un company_id, devuelva la cantidad de productos únicos asociados a esa empresa.
```sql
DELIMITER //

CREATE FUNCTION productos_por_empresa(company_id VARCHAR(20))
RETURNS INT
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE total_productos INT;

    SELECT COUNT(DISTINCT product_id) INTO total_productos
    FROM companyproducts
    WHERE company_id = company_id;

    RETURN total_productos;
END //

DELIMITER ;

SELECT productos_por_empresa('COMP1017');
```

14. Como gerente, deseo una función que retorne el nivel de actividad de un cliente (frecuente, esporádico, inactivo), según su número de calificaciones.
```sql
DELIMITER //

CREATE FUNCTION nivel_actividad(cliente_id INT)
RETURNS VARCHAR(20)
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE num_calificaciones INT;

    SELECT COUNT(*) INTO num_calificaciones
    FROM rates
    WHERE customer_id = cliente_id;

    IF num_calificaciones > 10 THEN
        RETURN 'Frecuente';
    ELSEIF num_calificaciones BETWEEN 4 AND 10 THEN
        RETURN 'Esporádico';
    ELSE
        RETURN 'Inactivo';
    END IF;
END //

DELIMITER ;

SELECT nivel_actividad(1);
```

15. Como administrador, quiero una función que calcule el precio promedio ponderado de un producto, tomando en cuenta su uso en favoritos.
```sql
DELIMITER //

CREATE FUNCTION precio_prom_ponderado(product_id INT)
RETURNS DOUBLE
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE total_ponderado DOUBLE;
    DECLARE total_favoritos INT;
    DECLARE precio DOUBLE;
    
    SELECT price INTO precio
    FROM companyproducts
    WHERE product_id = product_id
    LIMIT 1;  

    IF precio IS NULL THEN
        RETURN 0;
    END IF;
    

    SELECT COUNT(*) INTO total_favoritos
    FROM detail_favorites
    WHERE product_id = product_id;
    
    SET total_ponderado = precio * total_favoritos;

    IF total_favoritos > 0 THEN
        RETURN total_ponderado / total_favoritos;
    ELSE
        RETURN 0;
    END IF;

END //

DELIMITER ;


SELECT precio_prom_ponderado(1);
```

16. Como técnico, deseo una función que me indique si un benefit_id está asignado a más de una audiencia o membresía (valor booleano).
```sql
DELIMITER //

CREATE FUNCTION beneficio_asignado_multiple(benefit_id INT)
RETURNS BOOLEAN
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE total_asignaciones INT;
    
    SELECT COUNT(*) INTO total_asignaciones
    FROM (
        SELECT audience_id FROM audiencebenefits WHERE benefit_id = benefit_id
        UNION
        SELECT membership_id FROM membershipsbenefits WHERE benefit_id = benefit_id
    ) AS asignaciones;

    IF total_asignaciones > 1 THEN
        RETURN TRUE;
    ELSE
        RETURN FALSE;
    END IF;

END //

DELIMITER ;

SELECT beneficio_asignado_multiple(1);
```

17. Como cliente, quiero una función que, dada mi ciudad, retorne un índice de variedad basado en número de empresas y productos.
```sql
DELIMITER //

CREATE FUNCTION indice_variedad(city_id VARCHAR(15))
RETURNS DOUBLE
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE num_empresas INT;
    DECLARE num_productos INT;

    SELECT COUNT(DISTINCT cp.company_id) 
    INTO num_empresas
    FROM companies c
    JOIN companyproducts cp ON c.id = cp.company_id
    WHERE c.city_id = city_id;

    SELECT COUNT(DISTINCT cp.product_id)
    INTO num_productos
    FROM companies c
    JOIN companyproducts cp ON c.id = cp.company_id
    WHERE c.city_id = city_id;
  
    IF num_empresas > 0 THEN
        RETURN num_productos / num_empresas;
    ELSE
        RETURN 0;  
    END IF;
END //

DELIMITER ;


SELECT indice_variedad('05001');
```

18. Como gestor de calidad, deseo una función que evalúe si un producto debe ser desactivado por tener baja calificación histórica.
```sql
DELIMITER //

CREATE FUNCTION evaluar_desactivacion_producto(product_id INT)
RETURNS BOOLEAN
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE promedio_calificacion DOUBLE;
    DECLARE umbral DOUBLE DEFAULT 3.0;

    SELECT AVG(rating) INTO promedio_calificacion
    FROM rates
    WHERE product_id = product_id;
    
    IF promedio_calificacion < umbral THEN
        RETURN TRUE; 
    ELSE
        RETURN FALSE; 
    END IF;

END //

DELIMITER ;

SELECT evaluar_desactivacion_producto(1);
```

19. Como desarrollador, quiero una función que calcule el índice de popularidad de un producto (combinando favoritos y ratings).
```sql
DELIMITER //

CREATE FUNCTION indice_popularidad_product(product_id INT)
RETURNS DOUBLE
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE num_favoritos INT;
    DECLARE promedio_calificacion DOUBLE;
    DECLARE peso_favoritos DOUBLE DEFAULT 0.5;
    DECLARE peso_calificacion DOUBLE DEFAULT 0.5; 
    
    SELECT COUNT(*) INTO num_favoritos
    FROM detail_favorites
    WHERE product_id = product_id;
    
    SELECT AVG(rating) INTO promedio_calificacion
    FROM rates
    WHERE product_id = product_id;
    
    RETURN (num_favoritos * peso_favoritos) + (promedio_calificacion * peso_calificacion);
    
END //

DELIMITER ;

SELECT indice_popularidad_product(1);
```

20. Como auditor, deseo una función que genere un código único basado en el nombre del producto y su fecha de creación.
```sql
DELIMITER //

CREATE FUNCTION generar_codigo_producto(product_name VARCHAR(255), created_at DATETIME)
RETURNS VARCHAR(255)
DETERMINISTIC
BEGIN
    DECLARE code VARCHAR(255);

    SET code = CONCAT(product_name, '-', DATE_FORMAT(created_at, '%Y%m%d%H%i%s'));
    
    RETURN MD5(code);
END //

DELIMITER ;

SELECT generar_codigo_producto('Laptop', '2023-07-15 14:35:20');
```