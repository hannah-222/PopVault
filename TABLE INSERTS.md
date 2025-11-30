# Inserts For Creating Tables
--------------------------------

### Collectors:
#### stores information about Funko Pop collectors
```sql
CREATE TABLE collectors (
    collector_id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100),
    join_date DATE,
    total_collection_value DECIMAL(10,2) DEFAULT 0.00,
    INDEX idx_last_name (last_name)
) ENGINE=InnoDB
```

### Franchise:
#### Stores franchise information
```sql
CREATE TABLE franchises (
    franchise_id INT AUTO_INCREMENT PRIMARY KEY,
    franchise_name VARCHAR(100) NOT NULL
) ENGINE=InnoDB
```

### Funko Pops:
#### Stores details about individual Funko Pop figures
```sql
CREATE TABLE funko_pops (
    pop_id INT AUTO_INCREMENT PRIMARY KEY,
    pop_name VARCHAR(150) NOT NULL,
    pop_number VARCHAR(20),
    franchise_id INT NOT NULL,
    release_year YEAR,
    is_exclusive BOOLEAN NOT NULL,
    exclusive_retailer VARCHAR(100),
    rarity ENUM('Common', 'Uncommon', 'Rare', 'Chase', 'Grail') NOT NULL,
    estimated_value DECIMAL(8,2) NOT NULL,
    INDEX idx_franchise (franchise_id),
    INDEX idx_rarity (rarity),
    INDEX idx_pop_name (pop_name),
    FOREIGN KEY (franchise_id) REFERENCES franchises(franchise_id)
) ENGINE=InnoDB
```
### Vault:
#### Junction table linking collectors to their owned Funko Pops
```sql
CREATE TABLE vault (
    vault_id INT AUTO_INCREMENT PRIMARY KEY,
    collector_id INT NOT NULL,
    pop_id INT NOT NULL,
    purchase_date DATE,
    purchase_price DECIMAL(8,2),
    pop_condition ENUM('Mint', 'Near Mint', 'Good', 'Fair', 'Out of Box') DEFAULT 'Mint',
    storage_location VARCHAR(100),
    INDEX idx_collector (collector_id),
    INDEX idx_pop (pop_id),
    INDEX idx_purchase_date (purchase_date),
    FOREIGN KEY (collector_id) REFERENCES collectors(collector_id),
    FOREIGN KEY (pop_id) REFERENCES funko_pops(pop_id)
) ENGINE=InnoDB
```


