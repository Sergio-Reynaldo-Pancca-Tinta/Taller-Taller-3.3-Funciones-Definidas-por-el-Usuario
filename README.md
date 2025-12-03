#   INFORGRAFIA
#   https://sergio-reynaldo-pancca-tinta.github.io/Taller-Taller-3.3-Funciones-Definidas-por-el-Usuario/
Funciones Definidas por el Usuario
-- =============================================
-- CREACIÓN DE BASE DE DATOS: BibliotecaUniversitaria
-- =============================================
USE master;
GO

IF DB_ID('BibliotecaUniversitaria') IS NOT NULL
BEGIN
    ALTER DATABASE BibliotecaUniversitaria SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
    DROP DATABASE BibliotecaUniversitaria;
END
GO

CREATE DATABASE BibliotecaUniversitaria;
GO

USE BibliotecaUniversitaria;
GO

-- =============================================
-- TABLAS PRINCIPALES (10 TABLAS)
-- =============================================

-- Tabla 1: Usuarios
CREATE TABLE Usuarios (
    UsuarioID INT PRIMARY KEY IDENTITY(1,1),
    CodigoUsuario VARCHAR(20) UNIQUE NOT NULL,
    Cedula VARCHAR(13) UNIQUE NOT NULL,
    Nombre VARCHAR(50) NOT NULL,
    Apellido VARCHAR(50) NOT NULL,
    Email VARCHAR(100),
    Telefono VARCHAR(20),
    TipoUsuario VARCHAR(20) CHECK (TipoUsuario IN ('ESTUDIANTE', 'PROFESOR', 'ADMINISTRATIVO', 'VISITANTE')),
    Facultad VARCHAR(50),
    Carrera VARCHAR(50),
    Semestre INT,
    FechaRegistro DATE DEFAULT GETDATE(),
    Estado VARCHAR(20) DEFAULT 'ACTIVO' CHECK (Estado IN ('ACTIVO', 'INACTIVO', 'SUSPENDIDO')),
    MultaAcumulada DECIMAL(10,2) DEFAULT 0.00
);
GO

-- Tabla 2: Libros
CREATE TABLE Libros (
    LibroID INT PRIMARY KEY IDENTITY(1,1),
    ISBN VARCHAR(20) UNIQUE NOT NULL,
    Titulo VARCHAR(200) NOT NULL,
    Subtitulo VARCHAR(200),
    AutorPrincipal VARCHAR(100) NOT NULL,
    AutoresSecundarios VARCHAR(500),
    AnioPublicacion INT,
    Edicion VARCHAR(20),
    Editorial VARCHAR(100),
    Categoria VARCHAR(50),
    Subcategoria VARCHAR(50),
    Paginas INT,
    Idioma VARCHAR(30),
    Precio DECIMAL(10,2),
    CantidadCopias INT DEFAULT 1,
    CopiasDisponibles INT DEFAULT 1,
    FechaAdquisicion DATE,
    EstadoLibro VARCHAR(20) DEFAULT 'DISPONIBLE' CHECK (EstadoLibro IN ('DISPONIBLE', 'PRESTADO', 'RESERVADO', 'REPARACION', 'PERDIDO'))
);
GO

-- Tabla 3: Prestamos
CREATE TABLE Prestamos (
    PrestamoID INT PRIMARY KEY IDENTITY(1,1),
    LibroID INT NOT NULL FOREIGN KEY REFERENCES Libros(LibroID),
    UsuarioID INT NOT NULL FOREIGN KEY REFERENCES Usuarios(UsuarioID),
    FechaPrestamo DATE NOT NULL DEFAULT GETDATE(),
    FechaDevolucionEsperada DATE NOT NULL,
    FechaDevolucionReal DATE,
    DiasPrestamo INT DEFAULT 15,
    EstadoPrestamo VARCHAR(20) DEFAULT 'ACTIVO' CHECK (EstadoPrestamo IN ('ACTIVO', 'DEVUELTO', 'VENCIDO', 'PERDIDO')),
    MultaGenerada DECIMAL(10,2) DEFAULT 0.00,
    Observaciones VARCHAR(500)
);
GO

-- Tabla 4: Reservas
CREATE TABLE Reservas (
    ReservaID INT PRIMARY KEY IDENTITY(1,1),
    LibroID INT NOT NULL FOREIGN KEY REFERENCES Libros(LibroID),
    UsuarioID INT NOT NULL FOREIGN KEY REFERENCES Usuarios(UsuarioID),
    FechaReserva DATE NOT NULL DEFAULT GETDATE(),
    FechaVencimientoReserva DATE NOT NULL,
    EstadoReserva VARCHAR(20) DEFAULT 'PENDIENTE' CHECK (EstadoReserva IN ('PENDIENTE', 'APROBADA', 'CANCELADA', 'VENCIDA')),
    Prioridad INT DEFAULT 1
);
GO

-- Tabla 5: Multas
CREATE TABLE Multas (
    MultaID INT PRIMARY KEY IDENTITY(1,1),
    PrestamoID INT NOT NULL FOREIGN KEY REFERENCES Prestamos(PrestamoID),
    UsuarioID INT NOT NULL FOREIGN KEY REFERENCES Usuarios(UsuarioID),
    Monto DECIMAL(10,2) NOT NULL,
    FechaGeneracion DATE DEFAULT GETDATE(),
    FechaPago DATE,
    EstadoMulta VARCHAR(20) DEFAULT 'PENDIENTE' CHECK (EstadoMulta IN ('PENDIENTE', 'PAGADA', 'CANCELADA')),
    Motivo VARCHAR(100),
    DiasRetraso INT
);
GO

-- Tabla 6: Categorias
CREATE TABLE Categorias (
    CategoriaID INT PRIMARY KEY IDENTITY(1,1),
    NombreCategoria VARCHAR(50) NOT NULL,
    Descripcion VARCHAR(200),
    MaximoDiasPrestamo INT DEFAULT 15,
    MultaPorDia DECIMAL(5,2) DEFAULT 0.50
);
GO

-- Tabla 7: Editoriales
CREATE TABLE Editoriales (
    EditorialID INT PRIMARY KEY IDENTITY(1,1),
    NombreEditorial VARCHAR(100) NOT NULL,
    Pais VARCHAR(50),
    Email VARCHAR(100),
    Telefono VARCHAR(25)
);
GO

-- Tabla 8: Autores
CREATE TABLE Autores (
    AutorID INT PRIMARY KEY IDENTITY(1,1),
    Nombre VARCHAR(50) NOT NULL,
    Apellido VARCHAR(50) NOT NULL,
    Nacionalidad VARCHAR(50),
    FechaNacimiento DATE,
    Biografia VARCHAR(MAX)
);
GO

-- Tabla 9: LibrosAutores
CREATE TABLE LibrosAutores (
    LibroID INT NOT NULL FOREIGN KEY REFERENCES Libros(LibroID),
    AutorID INT NOT NULL FOREIGN KEY REFERENCES Autores(AutorID),
    TipoAutor VARCHAR(20) CHECK (TipoAutor IN ('PRINCIPAL', 'COAUTOR', 'TRADUCTOR', 'EDITOR')),
    PRIMARY KEY (LibroID, AutorID)
);
GO

-- Tabla 10: HistorialPrestamos
CREATE TABLE HistorialPrestamos (
    HistorialID INT PRIMARY KEY IDENTITY(1,1),
    LibroID INT NOT NULL,
    UsuarioID INT NOT NULL,
    FechaPrestamo DATE NOT NULL,
    FechaDevolucion DATE,
    DiasUtilizado INT,
    EstadoFinal VARCHAR(20)
);
GO

-- =============================================
-- DATOS DE EJEMPLO PARA LAS 10 TABLAS
-- =============================================

-- Insertar Usuarios
INSERT INTO Usuarios (CodigoUsuario, Cedula, Nombre, Apellido, Email, Telefono, TipoUsuario, Facultad, Carrera, Semestre) VALUES
('EST2023001', '1723456789', 'Juan', 'Pérez', 'juan.perez@universidad.edu.ec', '0987654321', 'ESTUDIANTE', 'Ingeniería', 'Sistemas', 5),
('EST2023002', '1729876543', 'María', 'Gómez', 'maria.gomez@universidad.edu.ec', '0991234567', 'ESTUDIANTE', 'Ciencias', 'Matemáticas', 7);
GO

-- Insertar Editoriales
INSERT INTO Editoriales (NombreEditorial, Pais, Email, Telefono) VALUES
('McGraw-Hill', 'Estados Unidos', 'contact@mcgraw-hill.com', '+1-212-904-2000'),
('Pearson', 'Reino Unido', 'support@pearson.com', '+44-20-7010-2000');
GO

-- Insertar Autores
INSERT INTO Autores (Nombre, Apellido, Nacionalidad, FechaNacimiento, Biografia) VALUES
('Robert', 'Martin', 'Estados Unidos', '1952-12-05', 'Conocido como Uncle Bob, experto en ingeniería de software'),
('Andrew', 'Hunt', 'Estados Unidos', '1964-09-09', 'Co-autor de The Pragmatic Programmer');
GO

-- Insertar Libros
INSERT INTO Libros (ISBN, Titulo, AutorPrincipal, AnioPublicacion, Editorial, Categoria, Paginas, Precio, CantidadCopias, CopiasDisponibles) VALUES
('978-0132350884', 'Clean Code: A Handbook of Agile Software Craftsmanship', 'Robert Martin', 2008, 'Prentice Hall', 'PROGRAMACION', 464, 45.99, 5, 3),
('978-0201616224', 'The Pragmatic Programmer', 'Andrew Hunt', 1999, 'Addison-Wesley', 'PROGRAMACION', 352, 39.99, 3, 1);
GO

-- Insertar Categorias
INSERT INTO Categorias (NombreCategoria, Descripcion, MaximoDiasPrestamo, MultaPorDia) VALUES
('PROGRAMACION', 'Libros de programación y desarrollo de software', 15, 0.50),
('MATEMATICAS', 'Libros de matemáticas y cálculo', 20, 0.30);
GO

-- Insertar LibrosAutores
INSERT INTO LibrosAutores (LibroID, AutorID, TipoAutor) VALUES
(1, 1, 'PRINCIPAL'),
(2, 2, 'PRINCIPAL');
GO

-- Insertar Prestamos
INSERT INTO Prestamos (LibroID, UsuarioID, FechaPrestamo, FechaDevolucionEsperada, FechaDevolucionReal, EstadoPrestamo) VALUES
(1, 1, '2024-01-10', '2024-01-25', '2024-01-22', 'DEVUELTO'),
(2, 2, '2024-02-01', '2024-02-16', NULL, 'VENCIDO');
GO

-- Insertar Multas
INSERT INTO Multas (PrestamoID, UsuarioID, Monto, FechaGeneracion, EstadoMulta, Motivo, DiasRetraso) VALUES
(2, 2, 5.50, '2024-02-17', 'PENDIENTE', 'Retraso en devolución', 11);
GO

-- Insertar HistorialPrestamos
INSERT INTO HistorialPrestamos (LibroID, UsuarioID, FechaPrestamo, FechaDevolucion, DiasUtilizado, EstadoFinal) VALUES
(1, 1, '2024-01-10', '2024-01-22', 12, 'DEVUELTO');
GO

PRINT '=== BASE DE DATOS BIBLIOTECAUNIVERSITARIA CREADA ===';
PRINT '=== 10 TABLAS CREADAS CON ÉXITO ===';
GO
