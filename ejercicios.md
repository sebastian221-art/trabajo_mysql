--NORMALIZACION 1

-- 1. Crear tabla HistorialPedidos
CREATE TABLE HistorialPedidos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    pedido_id INT,
    fecha_modificacion DATETIME,
    total_anterior DECIMAL(10,2),
    total_nuevo DECIMAL(10,2),
    FOREIGN KEY (pedido_id) REFERENCES Pedidos(id)
);

-- 2. Evaluar tabla Clientes hasta 3NF (ya está en 3NF si cada dato es atómico, email es único)

-- 3. Separar Empleados
CREATE TABLE Puestos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(50)
);

CREATE TABLE DatosEmpleados (
    id INT PRIMARY KEY,
    nombre VARCHAR(100),
    puesto_id INT,
    salario DECIMAL(10,2),
    fecha_contratacion DATE,
    FOREIGN KEY (puesto_id) REFERENCES Puestos(id)
);

-- 4. Relación Clientes y UbicacionCliente (ya normalizado, 1:n)

-- 5. Normalizar Proveedores
CREATE TABLE ContactoProveedores (
    id INT PRIMARY KEY AUTO_INCREMENT,
    proveedor_id INT,
    nombre_contacto VARCHAR(100),
    telefono VARCHAR(20),
    FOREIGN KEY (proveedor_id) REFERENCES Proveedores(id)
);

-- 6. Tabla Telefonos
CREATE TABLE TelefonosCliente (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cliente_id INT,
    telefono VARCHAR(20),
    FOREIGN KEY (cliente_id) REFERENCES Clientes(id)
);

-- 7. TiposProductos como jerarquía
ALTER TABLE TiposProductos ADD parent_id INT;
ALTER TABLE TiposProductos ADD FOREIGN KEY (parent_id) REFERENCES TiposProductos(id);

-- 8. Pedidos y DetallesPedido consistentes
-- Asegurar que DetallesPedido.precio = Productos.precio (se puede controlar por trigger)

-- 9. Relación muchos a muchos Empleados-Proveedores
CREATE TABLE EmpleadosProveedores (
    id INT PRIMARY KEY AUTO_INCREMENT,
    empleado_id INT,
    proveedor_id INT,
    FOREIGN KEY (empleado_id) REFERENCES Empleados(id),
    FOREIGN KEY (proveedor_id) REFERENCES Proveedores(id)
);

-- 10. UbicacionCliente como Ubicaciones genéricas
CREATE TABLE Ubicaciones (
    id INT PRIMARY KEY AUTO_INCREMENT,
    direccion VARCHAR(255),
    ciudad VARCHAR(100),
    estado VARCHAR(50),
    codigo_postal VARCHAR(10),
    pais VARCHAR(50)
);

ALTER TABLE UbicacionCliente ADD ubicacion_id INT;
ALTER TABLE UbicacionCliente DROP COLUMN direccion, DROP COLUMN ciudad,
DROP COLUMN estado, DROP COLUMN codigo_postal, DROP COLUMN pais;
ALTER TABLE UbicacionCliente ADD FOREIGN KEY (ubicacion_id) REFERENCES Ubicaciones(id);
 


--JOINS 2


-- 1. Pedidos y nombres de clientes
SELECT Pedidos.id, Clientes.nombre, Pedidos.fecha
FROM Pedidos
INNER JOIN Clientes ON Pedidos.cliente_id = Clientes.id;

-- 2. Productos y proveedores
SELECT Productos.nombre, Proveedores.nombre AS proveedor
FROM Productos
INNER JOIN Proveedores ON Productos.proveedor_id = Proveedores.id;

-- 3. Pedidos y ubicaciones
SELECT Pedidos.id, UbicacionCliente.direccion
FROM Pedidos
LEFT JOIN UbicacionCliente ON Pedidos.cliente_id = UbicacionCliente.cliente_id;

-- 4. Empleados con/sin pedidos
SELECT Empleados.nombre, Pedidos.id
FROM Empleados
LEFT JOIN Pedidos ON Empleados.id = Pedidos.cliente_id;

-- 5. Tipos y productos
SELECT TiposProductos.tipo_nombre, Productos.nombre
FROM TiposProductos
INNER JOIN Productos ON TiposProductos.id = Productos.tipo_id;

-- 6. Clientes y número de pedidos
SELECT Clientes.nombre, COUNT(Pedidos.id) AS num_pedidos
FROM Clientes
LEFT JOIN Pedidos ON Clientes.id = Pedidos.cliente_id
GROUP BY Clientes.id;

-- 7. Pedidos y empleados (suponiendo campo empleado_id en Pedidos)
ALTER TABLE Pedidos ADD empleado_id INT;
ALTER TABLE Pedidos ADD FOREIGN KEY (empleado_id) REFERENCES Empleados(id);
SELECT Pedidos.id, Empleados.nombre
FROM Pedidos
INNER JOIN Empleados ON Pedidos.empleado_id = Empleados.id;

-- 8. Productos no pedidos
SELECT Productos.nombre
FROM Productos
RIGHT JOIN DetallesPedido ON Productos.id = DetallesPedido.producto_id
WHERE DetallesPedido.producto_id IS NULL;

-- 9. Total pedidos y ubicación
SELECT Pedidos.id, Clientes.nombre, UbicacionCliente.direccion
FROM Pedidos
JOIN Clientes ON Pedidos.cliente_id = Clientes.id
JOIN UbicacionCliente ON Clientes.id = UbicacionCliente.cliente_id;

-- 10. Proveedores, productos y tipos
SELECT Proveedores.nombre AS proveedor, Productos.nombre AS producto, TiposProductos.tipo_nombre
FROM Productos
JOIN Proveedores ON Productos.proveedor_id = Proveedores.id
JOIN TiposProductos ON Productos.tipo_id = TiposProductos.id;


--CONSULTAS SIMPLES 3


-- 1. Productos > $50
SELECT * FROM Productos WHERE precio > 50;

-- 2. Clientes en ciudad específica
SELECT Clientes.* 
FROM Clientes 
JOIN UbicacionCliente ON Clientes.id = UbicacionCliente.cliente_id 
WHERE UbicacionCliente.ciudad = 'Bogotá';

-- 3. Empleados contratados en últimos 2 años
SELECT * FROM Empleados 
WHERE fecha_contratacion >= DATE_SUB(CURDATE(), INTERVAL 2 YEAR);

-- 4. Proveedores con más de 5 productos
SELECT proveedor_id, COUNT(*) AS total
FROM Productos
GROUP BY proveedor_id
HAVING total > 5;

-- 5. Clientes sin dirección
SELECT * FROM Clientes 
WHERE id NOT IN (
  SELECT cliente_id FROM UbicacionCliente
);

-- 6. Total ventas por cliente
SELECT cliente_id, SUM(total) AS total_ventas
FROM Pedidos
GROUP BY cliente_id;

-- 7. Salario promedio
SELECT AVG(salario) AS salario_promedio FROM Empleados;

-- 8. Tipos de productos
SELECT * FROM TiposProductos;

-- 9. 3 productos más caros
SELECT * FROM Productos ORDER BY precio DESC LIMIT 3;

-- 10. Cliente con más pedidos
SELECT cliente_id, COUNT(*) AS total
FROM Pedidos
GROUP BY cliente_id
ORDER BY total DESC
LIMIT 1;


--CONSULTAS MULTITABLA 4


-- 1. Pedidos y cliente
SELECT Pedidos.*, Clientes.nombre
FROM Pedidos
JOIN Clientes ON Pedidos.cliente_id = Clientes.id;

-- 2. Ubicación en pedidos
SELECT Pedidos.id, UbicacionCliente.direccion
FROM Pedidos
JOIN UbicacionCliente ON Pedidos.cliente_id = UbicacionCliente.cliente_id;

-- 3. Productos con proveedor y tipo
SELECT Productos.nombre, Proveedores.nombre, TiposProductos.tipo_nombre
FROM Productos
JOIN Proveedores ON Productos.proveedor_id = Proveedores.id
JOIN TiposProductos ON Productos.tipo_id = TiposProductos.id;

-- 4. Empleados gestionando pedidos en ciudad específica
SELECT Empleados.nombre
FROM Pedidos
JOIN Empleados ON Pedidos.empleado_id = Empleados.id
JOIN Clientes ON Pedidos.cliente_id = Clientes.id
JOIN UbicacionCliente ON Clientes.id = UbicacionCliente.cliente_id
WHERE UbicacionCliente.ciudad = 'Bogotá';

-- 5. 5 productos más vendidos
SELECT producto_id, SUM(cantidad) AS total_vendido
FROM DetallesPedido
GROUP BY producto_id
ORDER BY total_vendido DESC
LIMIT 5;

-- 6. Cantidad de pedidos por cliente y ciudad
SELECT Clientes.nombre, UbicacionCliente.ciudad, COUNT(Pedidos.id) AS total
FROM Pedidos
JOIN Clientes ON Pedidos.cliente_id = Clientes.id
JOIN UbicacionCliente ON Clientes.id = UbicacionCliente.cliente_id
GROUP BY Clientes.id, UbicacionCliente.ciudad;

-- 7. Clientes y proveedores en misma ciudad
SELECT DISTINCT Clientes.nombre AS cliente, Proveedores.nombre AS proveedor
FROM Clientes
JOIN UbicacionCliente ON Clientes.id = UbicacionCliente.cliente_id
JOIN Proveedores ON UbicacionCliente.ciudad = Proveedores.direccion;

-- 8. Total ventas por tipo
SELECT TiposProductos.tipo_nombre, SUM(DetallesPedido.cantidad * DetallesPedido.precio) AS total
FROM DetallesPedido
JOIN Productos ON DetallesPedido.producto_id = Productos.id
JOIN TiposProductos ON Productos.tipo_id = TiposProductos.id
GROUP BY TiposProductos.tipo_nombre;

-- 9. Empleados gestionando productos de un proveedor
SELECT DISTINCT Empleados.nombre
FROM Pedidos
JOIN Empleados ON Pedidos.empleado_id = Empleados.id
JOIN DetallesPedido ON Pedidos.id = DetallesPedido.pedido_id
JOIN Productos ON DetallesPedido.producto_id = Productos.id
WHERE Productos.proveedor_id = 1;

-- 10. Ingreso total por proveedor
SELECT Proveedores.nombre, SUM(DetallesPedido.cantidad * DetallesPedido.precio) AS ingreso
FROM DetallesPedido
JOIN Productos ON DetallesPedido.producto_id = Productos.id
JOIN Proveedores ON Productos.proveedor_id = Proveedores.id
GROUP BY Proveedores.id;

--SUBCONSULTAS 5


-- 1. Producto más caro por categoría
SELECT * FROM Productos p
WHERE precio = (
  SELECT MAX(precio) FROM Productos WHERE tipo_id = p.tipo_id
);

-- 2. Cliente con mayor total en pedidos
SELECT cliente_id, SUM(total) AS total
FROM Pedidos
GROUP BY cliente_id
ORDER BY total DESC
LIMIT 1;

-- 3. Empleados con salario > promedio
SELECT * FROM Empleados
WHERE salario > (
  SELECT AVG(salario) FROM Empleados
);

-- 4. Productos pedidos más de 5 veces
SELECT producto_id
FROM DetallesPedido
GROUP BY producto_id
HAVING COUNT(*) > 5;

-- 5. Pedidos > promedio
SELECT * FROM Pedidos
WHERE total > (
  SELECT AVG(total) FROM Pedidos
);

-- 6. 3 proveedores con más productos
SELECT proveedor_id, COUNT(*) AS total
FROM Productos
GROUP BY proveedor_id
ORDER BY total DESC
LIMIT 3;

-- 7. Productos con precio > promedio en su tipo
SELECT * FROM Productos p
WHERE precio > (
  SELECT AVG(precio) FROM Productos WHERE tipo_id = p.tipo_id
);

-- 8. Clientes con más pedidos que la media
SELECT cliente_id, COUNT(*) AS num
FROM Pedidos
GROUP BY cliente_id
HAVING num > (
  SELECT AVG(cantidad) FROM (
    SELECT COUNT(*) AS cantidad FROM Pedidos GROUP BY cliente_id
  ) AS temp
);

-- 9. Productos > precio promedio general
SELECT * FROM Productos
WHERE precio > (
  SELECT AVG(precio) FROM Productos
);

-- 10. Empleados con salario < promedio del departamento (puesto)
-- Requiere campo departamento o tabla de puestos con salario promedio


--PROCEDIMIENTOS ALMACENADOS 6


-- 1. Actualizar precio productos por proveedor
DELIMITER //
CREATE PROCEDURE ActualizarPrecioProveedor(IN prov_id INT, IN nuevo_precio DECIMAL(10,2))
BEGIN
  UPDATE Productos SET precio = nuevo_precio WHERE proveedor_id = prov_id;
END;
//
DELIMITER ;

-- 2. Dirección por ID de cliente
DELIMITER //
CREATE PROCEDURE ObtenerDireccionCliente(IN id_cliente INT)
BEGIN
  SELECT * FROM UbicacionCliente WHERE cliente_id = id_cliente;
END;
//
DELIMITER ;

-- 3. Registrar pedido
DELIMITER //
CREATE PROCEDURE RegistrarPedido(
  IN id_cliente INT,
  IN fecha DATE,
  IN total DECIMAL(10,2)
)
BEGIN
  INSERT INTO Pedidos(cliente_id, fecha, total) VALUES (id_cliente, fecha, total);
END;
//
DELIMITER ;

-- 4. Calcular total de ventas cliente
DELIMITER //
CREATE PROCEDURE TotalVentasCliente(IN id_cliente INT)
BEGIN
  SELECT SUM(total) FROM Pedidos WHERE cliente_id = id_cliente;
END;
//
DELIMITER ;

-- 5. Obtener empleados por puesto
DELIMITER //
CREATE PROCEDURE EmpleadosPorPuesto(IN nombre_puesto VARCHAR(50))
BEGIN
  SELECT * FROM Empleados WHERE puesto = nombre_puesto;
END;
//
DELIMITER ;

-- 6. Actualizar salario por puesto
DELIMITER //
CREATE PROCEDURE ActualizarSalario(IN puesto_nombre VARCHAR(50), IN nuevo_salario DECIMAL(10,2))
BEGIN
  UPDATE Empleados SET salario = nuevo_salario WHERE puesto = puesto_nombre;
END;
//
DELIMITER ;

-- 7. Pedidos entre fechas
DELIMITER //
CREATE PROCEDURE PedidosEntreFechas(IN fecha1 DATE, IN fecha2 DATE)
BEGIN
  SELECT * FROM Pedidos WHERE fecha BETWEEN fecha1 AND fecha2;
END;
//
DELIMITER ;

-- 8. Descuento a categoría
DELIMITER //
CREATE PROCEDURE AplicarDescuentoTipo(IN tipo INT, IN porcentaje DECIMAL(5,2))
BEGIN
  UPDATE Productos SET precio = precio - (precio * porcentaje / 100)
  WHERE tipo_id = tipo;
END;
//
DELIMITER ;

-- 9. Proveedores por tipo
DELIMITER //
CREATE PROCEDURE ProveedoresPorTipo(IN tipo INT)
BEGIN
  SELECT DISTINCT Proveedores.*
  FROM Productos
  JOIN Proveedores ON Productos.proveedor_id = Proveedores.id
  WHERE Productos.tipo_id = tipo;
END;
//
DELIMITER ;

-- 10. Pedido de mayor valor
DELIMITER //
CREATE PROCEDURE PedidoMayorValor()
BEGIN
  SELECT * FROM Pedidos ORDER BY total DESC LIMIT 1;
END;
//
DELIMITER ;
