/*
================================================================================
PROJE ADI: NovaStore E-Ticaret Veri Tabanı Tasarımı
HAZIRLAYAN: Koray KILIÇ
DOSYA: Koray_NovaStore_Proje.sql
TANIM: Bu script, NovaStoreDB veri tabanını oluşturur, tabloları kurar, 
       örnek verileri ekler, istenen raporları sorgular ve yedek alır.
================================================================================
*/

-- 1. VERİ TABANI OLUŞTURMA
-- Eğer veritabanı zaten varsa hata vermesin diye kontrol ediyoruz
USE master;
GO

IF NOT EXISTS (SELECT * FROM sys.databases WHERE name = 'NovaStoreDB')
BEGIN
    CREATE DATABASE NovaStoreDB;
END
GO

USE NovaStoreDB;
GO

-- 2. TABLO ŞEMALARININ OLUŞTURULMASI

-- Kategoriler Tablosu
IF NOT EXISTS (SELECT * FROM sysobjects WHERE name='Categories' AND xtype='U')
CREATE TABLE Categories (
    CategoryID INT PRIMARY KEY IDENTITY(1,1),
    CategoryName VARCHAR(50) NOT NULL
);

-- Müşteriler Tablosu
IF NOT EXISTS (SELECT * FROM sysobjects WHERE name='Customers' AND xtype='U')
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY IDENTITY(1,1),
    FullName VARCHAR(50),
    City VARCHAR(20),
    Email VARCHAR(100) UNIQUE
);

-- Ürünler Tablosu
IF NOT EXISTS (SELECT * FROM sysobjects WHERE name='Products' AND xtype='U')
CREATE TABLE Products (
    ProductID INT PRIMARY KEY IDENTITY(1,1),
    ProductName VARCHAR(100) NOT NULL,
    Price DECIMAL(10,2),
    Stock INT DEFAULT 0,
    CategoryID INT,
    CONSTRAINT FK_Products_Categories FOREIGN KEY (CategoryID) REFERENCES Categories(CategoryID)
);

-- Siparişler Tablosu
IF NOT EXISTS (SELECT * FROM sysobjects WHERE name='Orders' AND xtype='U')
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY IDENTITY(1,1),
    CustomerID INT,
    OrderDate DATETIME DEFAULT GETDATE(),
    TotalAmount DECIMAL(10,2),
    CONSTRAINT FK_Orders_Customers FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);

-- Sipariş Detayları Tablosu
IF NOT EXISTS (SELECT * FROM sysobjects WHERE name='OrderDetails' AND xtype='U')
CREATE TABLE OrderDetails (
    DetailID INT PRIMARY KEY IDENTITY(1,1),
    OrderID INT,
    ProductID INT,
    Quantity INT,
    CONSTRAINT FK_Details_Orders FOREIGN KEY (OrderID) REFERENCES Orders(OrderID),
    CONSTRAINT FK_Details_Products FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
);
GO

-- 3. ÖRNEK VERİLERİN EKLENMESİ (INSERT)

-- Kategoriler
INSERT INTO Categories (CategoryName) VALUES 
('Elektronik'), ('Giyim'), ('Kitap'), ('Ev & Yaşam');

-- Ürünler
INSERT INTO Products (ProductName, Price, Stock, CategoryID) VALUES 
('Laptop', 25000.00, 10, 1),
('Akıllı Telefon', 15000.00, 20, 1),
('Kot Pantolon', 800.00, 50, 2),
('Tişört', 300.00, 100, 2),
('SQL Eğitimi Kitabı', 250.00, 30, 3),
('Kahve Makinesi', 3000.00, 5, 4);

-- Müşteriler
INSERT INTO Customers (FullName, City, Email) VALUES 
('Ahmet Yılmaz', 'İstanbul', 'ahmet@test.com'),
('Ayşe Demir', 'Ankara', 'ayse@test.com'),
('Mehmet Kaya', 'İzmir', 'mehmet@test.com');

-- Siparişler
INSERT INTO Orders (CustomerID, OrderDate, TotalAmount) VALUES 
(1, '2023-10-01', 25300.00),
(2, '2023-10-05', 800.00),
(1, GETDATE(), 3000.00);

-- Sipariş Detayları
INSERT INTO OrderDetails (OrderID, ProductID, Quantity) VALUES 
(1, 1, 1),
(1, 4, 1),
(2, 3, 1),
(3, 6, 1);
GO

-- 4. RAPORLAMA VE SORGULAR

-- Soru 1: Stok miktarı 20'den az olan ürünler
SELECT ProductName, Stock, Price 
FROM Products 
WHERE Stock < 20 
ORDER BY Stock ASC;

-- Soru 2: Müşteri Sipariş Listesi (JOIN)
SELECT C.FullName, C.City, O.OrderDate, O.TotalAmount
FROM Customers C
INNER JOIN Orders O ON C.CustomerID = O.CustomerID;

-- Soru 3: Her Müşterinin Toplam Harcaması (GROUP BY)
SELECT C.FullName, SUM(O.TotalAmount) AS ToplamHarcama
FROM Customers C
JOIN Orders O ON C.CustomerID = O.CustomerID
GROUP BY C.FullName
ORDER BY ToplamHarcama DESC;

-- Soru 4: Kategori Bazlı Ürün Sayıları (LEFT JOIN)
SELECT C.CategoryName, COUNT(P.ProductID) AS UrunSayisi
FROM Categories C
LEFT JOIN Products P ON C.CategoryID = P.CategoryID
GROUP BY C.CategoryName;

-- Soru 5: Siparişlerin Üzerinden Geçen Gün Sayısı
SELECT OrderID, OrderDate, DATEDIFF(DAY, OrderDate, GETDATE()) AS GecenGun
FROM Orders;
GO

-- 5. VIEW OLUŞTURMA (Sanal Tablo)
-- Daha önce varsa silip yeniden oluşturalım
IF OBJECT_ID('vw_SiparisOzet', 'V') IS NOT NULL
    DROP VIEW vw_SiparisOzet;
GO

CREATE VIEW vw_SiparisOzet AS
SELECT 
    C.FullName AS MusteriAdi,
    O.OrderDate AS SiparisTarihi,
    P.ProductName AS UrunAdi,
    OD.Quantity AS Adet
FROM Customers C
INNER JOIN Orders O ON C.CustomerID = O.CustomerID
INNER JOIN OrderDetails OD ON O.OrderID = OD.OrderID
INNER JOIN Products P ON OD.ProductID = P.ProductID;
GO

-- View Testi
SELECT * FROM vw_SiparisOzet;
GO

-- 6. VERİ TABANI YEDEĞİ (BACKUP)
-- Not: C:\Yedek klasörünün var olduğundan emin olunmalıdır.
BACKUP DATABASE NovaStoreDB 
TO DISK = 'C:\Yedek\NovaStoreDB.bak'
WITH FORMAT, MEDIANAME = 'SQLServerBackups', NAME = 'Full Backup of NovaStoreDB';
GO




1. PROJE ÖZETİ
Bu proje kapsamında, "NovaStore" isimli e-ticaret platformunun ihtiyaç duyduğu ilişkisel veri tabanı (RDBMS) SQL Server üzerinde tasarlanmıştır. Veri bütünlüğünü sağlamak adına Primary Key ve Foreign Key kısıtlamaları (Constraints) kullanılmıştır.
2. VERİ TABANI ŞEMASI VE TABLOLAR
Proje kapsamında aşağıdaki tablolar oluşturulmuş ve ilişkilendirilmiştir:
•	Categories: Ürün kategorilerini tutar.
•	Products: Ürün bilgilerini ve stok durumunu tutar (Kategori ile ilişkilidir).
•	Customers: Müşteri bilgilerini saklar.
•	Orders: Sipariş başlık bilgilerini ve tarihini tutar (Müşteri ile ilişkilidir).
•	OrderDetails: Siparişin içindeki ürün detaylarını ve adetleri tutar.
3. GERÇEKLEŞTİRİLEN İŞLEMLER
•	DDL İşlemleri: CREATE komutları ile veritabanı ve tablo inşası tamamlandı.
•	DML İşlemleri: INSERT komutları ile kategoriler, ürünler ve müşteriler için örnek veri girişleri yapıldı.
•	Raporlama: JOIN, GROUP BY ve DATEDIFF fonksiyonları kullanılarak yönetimsel raporlar (Stok takibi, ciro analizi vb.) oluşturuldu.
•	View: Sık kullanılan sorgular için 'vw_SiparisOzet' isimli VIEW (Sanal Tablo) oluşturuldu.
•	Yedekleme: Veri tabanının .bak uzantılı yedeği (Backup) alındı.
