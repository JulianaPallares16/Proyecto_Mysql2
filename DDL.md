```sql
CREATE TABLE IF NOT EXISTS countries(
    isocode VARCHAR(6) PRIMARY KEY,
    name VARCHAR(50) UNIQUE,
    alfaisotwo VARCHAR(2) UNIQUE,
    alfaisothree VARCHAR(3) UNIQUE
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS subdivisioncategories(
    id INT AUTO_INCREMENT  PRIMARY KEY,
    description VARCHAR(40) UNIQUE
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS stateorregions(
    code VARCHAR(6) PRIMARY KEY,
    name VARCHAR(60) UNIQUE,
    country_id VARCHAR(6),
    CONSTRAINT FK_country_id FOREIGN KEY (country_id) REFERENCES countries(isocode),
    code3166 VARCHAR(10) UNIQUE,
    subdivision_id INT,
    CONSTRAINT FK_subdivision_id FOREIGN KEY (subdivision_id) REFERENCES subdivisioncategories(id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS citiesormunicipalities(
    code VARCHAR(10) PRIMARY KEY,
    name VARCHAR(60),
    statereg_id VARCHAR(6),
    CONSTRAINT FK_statereg_id FOREIGN KEY (statereg_id) REFERENCES stateorregions(code)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS categories(
    id INT AUTO_INCREMENT PRIMARY KEY,
    description VARCHAR(60) UNIQUE
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS audiences(
    id INT AUTO_INCREMENT PRIMARY KEY,
    description VARCHAR(60)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS typesidentifications(
    id INT AUTO_INCREMENT PRIMARY KEY,
    description VARCHAR(60) UNIQUE,
    sufix VARCHAR(5) UNIQUE
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS companies(
    id VARCHAR(20) PRIMARY KEY,
    type_id INT,
    CONSTRAINT FK_type_id FOREIGN KEY (type_id) REFERENCES typesidentifications(id),
    name VARCHAR(80),
    category_id INT,
    CONSTRAINT FK_category_id FOREIGN KEY (category_id) REFERENCES categories(id),
    city_id VARCHAR(10),
    CONSTRAINT FK_city_id FOREIGN KEY (city_id) REFERENCES citiesormunicipalities(code),
    audience_id INT,
    CONSTRAINT FK_audience_id FOREIGN KEY (audience_id) REFERENCES audiences(id),
    cellphone VARCHAR(15) UNIQUE,
    email VARCHAR(80) UNIQUE
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS memberships(
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) UNIQUE,
    description TEXT
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS periods(
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) UNIQUE
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS membershipsperiods(
    membership_id INT,
    CONSTRAINT FK_membership_id FOREIGN KEY (membership_id) REFERENCES memberships(id),
    period_id INT,
    CONSTRAINT FK_period_id FOREIGN KEY (period_id) REFERENCES periods(id),
    price DOUBLE,
    PRIMARY KEY (membership_id, period_id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS benefits(
    id INT AUTO_INCREMENT PRIMARY KEY,
    description VARCHAR(80),
    detail TEXT
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS audiencebenefits(
    audience_id INT,
    CONSTRAINT FK_audienc_id FOREIGN KEY (audience_id) REFERENCES audiences(id),
    benefit_id INT,
    CONSTRAINT FK_benefit_id FOREIGN KEY (benefit_id) REFERENCES benefits(id),
    PRIMARY KEY (audience_id, benefit_id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS membershipsbenefits(
    membership_id INT,
    CONSTRAINT FK_membershi_id FOREIGN KEY (membership_id) REFERENCES memberships(id),
    period_id INT,
    CONSTRAINT FK_perio_id FOREIGN KEY (period_id) REFERENCES periods(id),
    audience_id INT,
    CONSTRAINT FK_audien_id FOREIGN KEY (audience_id) REFERENCES audiences(id),
    benefit_id INT,
    CONSTRAINT FK_benefits_id FOREIGN KEY (benefit_id) REFERENCES benefits(id),
    PRIMARY KEY (membership_id, period_id, audience_id, benefit_id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS categories_polls(
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(80) UNIQUE
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS polls(
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(80) UNIQUE,
    description TEXT,
    isactive BOOLEAN,
    categorypoll_id INT,
    CONSTRAINT FK_categorypoll_id FOREIGN KEY (categorypoll_id) REFERENCES categories_polls(id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS unitofmeasure(
    id INT AUTO_INCREMENT PRIMARY KEY,
    description VARCHAR(60) UNIQUE
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS products(
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(60) UNIQUE,
    detail TEXT,
    price DOUBLE,
    category_id INT,
    CONSTRAINT FK_catego_id FOREIGN KEY (category_id) REFERENCES categories(id),
    image VARCHAR(80)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS companyproducts(
   company_id VARCHAR(20),
   CONSTRAINT FK_company_id FOREIGN KEY (company_id) REFERENCES companies(id),
   product_id INT,
   CONSTRAINT FK_product_id FOREIGN KEY (product_id) REFERENCES products(id),
   price DOUBLE,
   unitimeasure_id INT,
   CONSTRAINT FK_unitimeasure_id FOREIGN KEY (unitimeasure_id) REFERENCES unitofmeasure(id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS customers(
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(80),
    city_id VARCHAR(10),
    CONSTRAINT FK_cit_id FOREIGN KEY (city_id) REFERENCES citiesormunicipalities(code),
    audience_id INT,
    CONSTRAINT FK_audiency_id FOREIGN KEY (audience_id) REFERENCES audiences(id),
    cellphone VARCHAR(20),
    email VARCHAR(100),
    address VARCHAR(120)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS rates(
   customer_id INT,
   CONSTRAINT FK_customer_id FOREIGN KEY (customer_id) REFERENCES customers(id),
   company_id VARCHAR(20),
   CONSTRAINT FK_compani_id FOREIGN KEY (company_id) REFERENCES companies(id),
   poll_id INT,
   CONSTRAINT FK_poll_id FOREIGN KEY (poll_id) REFERENCES polls(id),
   daterating DATETIME,
   rating DOUBLE,
   PRIMARY KEY(customer_id, company_id, poll_id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS quality_products(
   product_id INT,
   CONSTRAINT FK_producto_id FOREIGN KEY (product_id) REFERENCES products(id),
   customer_id INT,
   CONSTRAINT FK_costomer_id FOREIGN KEY (customer_id) REFERENCES customers(id),
   poll_id INT,
   CONSTRAINT FK_pol_id FOREIGN KEY (poll_id) REFERENCES polls(id),
   company_id VARCHAR(20),
   CONSTRAINT FK_compan_id FOREIGN KEY (company_id) REFERENCES companies(id),
   daterating DATETIME,
   rating DOUBLE,
   PRIMARY KEY(customer_id, company_id, poll_id, product_id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS favorites(
   id INT AUTO_INCREMENT PRIMARY KEY,
   customer_id INT,
   CONSTRAINT FK_cstomer_id FOREIGN KEY (customer_id) REFERENCES customers(id),
   company_id VARCHAR(20),
   CONSTRAINT FK_companies_id FOREIGN KEY (company_id) REFERENCES companies(id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS detail_favorites(
   id INT AUTO_INCREMENT PRIMARY KEY,
   favorite_id INT,
   CONSTRAINT FK_favorite_id FOREIGN KEY (favorite_id) REFERENCES favorites(id),
   product_id INT,
   CONSTRAINT FK_products_id FOREIGN KEY (product_id) REFERENCES products(id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS customers_memberships(
    customer_id INT,
    CONSTRAINT FK_customer_id_membership FOREIGN KEY (customer_id) REFERENCES customers(id),
    membership_id INT,
    CONSTRAINT FK_membership_id_customers FOREIGN KEY (membership_id) REFERENCES memberships(id),
    period_id INT,
    CONSTRAINT FK_period_id_customers FOREIGN KEY (period_id) REFERENCES periods(id),
    start_date DATE,
    end_date DATE,
    PRIMARY KEY (customer_id, membership_id, period_id)
) ENGINE = INNODB;

ALTER TABLE companyproducts
ADD COLUMN is_available BOOLEAN NOT NULL DEFAULT TRUE;

ALTER TABLE customers
ADD COLUMN membership_active BOOLEAN NOT NULL,
ADD COLUMN is_active BOOLEAN NOT NULL;

ALTER TABLE products
ADD average_rating DECIMAL(3,2) DEFAULT NULL;

CREATE TABLE IF NOT EXISTS errores_log (
    id INT AUTO_INCREMENT PRIMARY KEY,
    descripcion TEXT,
    fecha_error DATETIME DEFAULT CURRENT_TIMESTAMP
);

ALTER TABLE customers_memberships
ADD COLUMN payment_confirmed BOOLEAN DEFAULT FALSE,
ADD COLUMN status VARCHAR(20) DEFAULT 'INACTIVA';

CREATE TABLE IF NOT EXISTS poll_questions (
    id INT AUTO_INCREMENT PRIMARY KEY,
    poll_id INT NOT NULL,
    question_text TEXT NOT NULL,
    CONSTRAINT FK_polls_id FOREIGN KEY (poll_id) REFERENCES polls(id)
) ENGINE = INNODB;

ALTER TABLE detail_favorites
ADD COLUMN created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP;

CREATE TABLE IF NOT EXISTS historial_precios (
    id INT AUTO_INCREMENT PRIMARY KEY,
    company_id VARCHAR(20) NOT NULL,
    product_id INT NOT NULL,
    old_price DOUBLE NOT NULL,
    new_price DOUBLE NOT NULL,
    changed_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT FK_hist_company FOREIGN KEY (company_id) REFERENCES companies(id),
    CONSTRAINT FK_hist_product FOREIGN KEY (product_id) REFERENCES products(id)
) ENGINE=INNODB;

ALTER TABLE polls
ADD COLUMN start_date DATETIME DEFAULT CURRENT_TIMESTAMP;

ALTER TABLE products
ADD COLUMN updated_at DATETIME;

CREATE TABLE IF NOT EXISTS log_acciones (
    id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT,
    CONSTRAINT FK_log_customer FOREIGN KEY (customer_id) REFERENCES customers(id),
    company_id VARCHAR(20),
    CONSTRAINT FK_log_company FOREIGN KEY (company_id) REFERENCES companies(id),
    poll_id INT,
    CONSTRAINT FK_log_poll FOREIGN KEY (poll_id) REFERENCES polls(id),
    daterating DATETIME,
    rating DOUBLE,
    accion VARCHAR(100),
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=INNODB;

CREATE TABLE IF NOT EXISTS notificaciones (
    id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT NOT NULL,
    message VARCHAR(255) NOT NULL,
    created_at DATETIME NOT NULL DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS bitacora (
    id INT AUTO_INCREMENT PRIMARY KEY,
    tabla VARCHAR(50),
    mensaje TEXT,
    fecha DATETIME DEFAULT CURRENT_TIMESTAMP
) ENGINE=INNODB;

ALTER TABLE companies
ADD COLUMN updated_at DATETIME DEFAULT CURRENT_TIMESTAMP;

ALTER TABLE polls
ADD COLUMN status VARCHAR(20) NOT NULL DEFAULT 'inactiva';

CREATE TABLE IF NOT EXISTS poll_status_log (
    id INT AUTO_INCREMENT PRIMARY KEY,
    poll_id INT NOT NULL,
    old_status VARCHAR(20),
    new_status VARCHAR(20),
    changed_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    changed_by VARCHAR(50), 
    CONSTRAINT FK_poll_log_poll FOREIGN KEY (poll_id) REFERENCES polls(id)
) ENGINE=INNODB;

CREATE TABLE IF NOT EXISTS product_metrics (
    product_id INT PRIMARY KEY,
    average_rating DOUBLE,
    updated_at DATETIME
);

CREATE TABLE IF NOT EXISTS products_backup LIKE products;

CREATE TABLE IF NOT EXISTS rates_backup LIKE rates;

CREATE TABLE IF NOT EXISTS recordatorios (
    customer_id INT,
    product_id INT,
    mensaje VARCHAR(255),
    fecha_recordatorio DATETIME DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (customer_id, product_id)
);

CREATE TABLE IF NOT EXISTS errores_log (
    id INT AUTO_INCREMENT PRIMARY KEY,
    descripcion TEXT,
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

ALTER TABLE benefits
ADD COLUMN created_at DATETIME DEFAULT CURRENT_TIMESTAMP;

```