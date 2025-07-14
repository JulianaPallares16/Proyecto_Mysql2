```sql
CREATE TABLE IF NOT EXISTS countries(
    isocode VARCHAR(6) PRIMARY KEY,
    name VARCHAR(50) UNIQUE,
    alfaisotwo VARCHAR(2) UNIQUE,
    alfaisothree VARCHAR(2) UNIQUE
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
    code VARCHAR(6) PRIMARY KEY,
    name VARCHAR(6) UNIQUE,
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
    sufix VARCHAR(60) UNIQUE
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS companies(
    id VARCHAR(20) PRIMARY KEY,
    type_id INT,
    CONSTRAINT FK_type_id FOREIGN KEY (type_id) REFERENCES typesidentifications(id),
    name VARCHAR(80),
    category_id INT,
    CONSTRAINT FK_category_id FOREIGN KEY (category_id) REFERENCES categories(id),
    city_id VARCHAR(6),
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