# Kimia_Farma_Big_Data_Analytics

---datamart---
CREATE TABLE kimia_farma.datamart_kf AS
SELECT
    tr.transaction_id,
    tr.date,
    tr.branch_id,
    kc.branch_name,
    kc.kota,
    kc.provinsi,
    kc.rating AS rating_cabang,
    tr.customer_name,
    pr.product_id,
    pr.product_name,
    tr.price AS actual_price,
    tr.discount_percentage,
    CASE
        WHEN tr.price <= 50000 THEN 0.1
        WHEN tr.price > 50000 - 100000 THEN 0.15
        WHEN tr.price > 100000 - 300000 THEN 0.2
        WHEN tr.price > 300000 - 500000 THEN 0.25
        When tr.price > 500000 THEN 0.30
        ELSE 0.3
    END AS persentase_gross_laba,
    tr.price * (1 - tr.discount_percentage) AS nett_sales,
    (tr.price * (1 - tr.discount_percentage) * 
        CASE
            WHEN tr.price <= 50000 THEN 0.1
            WHEN tr.price > 50000 - 100000 THEN 0.15
            WHEN tr.price > 100000 - 300000 THEN 0.2
            WHEN tr.price > 300000 - 500000 THEN 0.25
            WHEN tr.price > 500000 THEN 0.30
            ELSE 0.3
        END) AS nett_profit,
    tr.rating AS rating_transaksi
FROM
    kimia_farma.transaction AS tr
LEFT JOIN
    kimia_farma.cabang AS kc ON tr.branch_id = kc.branch_id
LEFT JOIN
    kimia_farma.product AS pr ON tr.product_id = pr.product_id
;

---profit per year---
CREATE TABLE kimia_farma.profit_year AS
SELECT
    EXTRACT(YEAR FROM kf.date) AS tahun,
    SUM(nett_sales) AS total_pendapatan,
    AVG(nett_sales) AS average_pendapatan
FROM
    kimia_farma.datamart_kf AS kf
GROUP BY
    tahun
ORDER BY
    tahun
;

----nett sales per branch province---
CREATE TABLE kimia_farma.nettsales_branch AS
SELECT 
    provinsi,
    COUNT(*) AS total_produk_terjual,
    SUM(nett_sales) AS nett_sales_branch
FROM 
    kimia_farma.datamart_kf
GROUP BY 
    provinsi
ORDER BY 
    nett_sales_branch DESC
LIMIT 10
;

----branch transaction rating---
CREATE TABLE kimia_farma.branch_transaction_rating AS
SELECT 
    branch_name, 
    kota,
    AVG(rating_cabang) AS ave_branch_rating,
    AVG(rating_transaksi) AS ave_transaction_rating
FROM 
    kimia_farma.datamart_kf
GROUP BY 
    branch_name, kota
ORDER BY 
    ave_branch_rating DESC, ave_transaction_rating ASC
LIMIT 10
;

----most transaction customer---
CREATE TABLE kimia_farma.mosttransac_customer AS
SELECT 
    customer_name,
    COUNT(transaction_id) AS total_transaction
FROM 
    kimia_farma.datamart_kf
GROUP BY 
    customer_name
ORDER BY 
    total_transaction DESC
LIMIT 10
;
