
### **1. 栏目表（categories）**
用于存储多级栏目，可以自关联形成层级结构。

```sql
CREATE TABLE categories (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '栏目ID',
    site_id INT NOT NULL COMMENT '所属站点ID',
    parent_id INT DEFAULT NULL COMMENT '父级栏目ID，NULL表示顶级栏目',
    name VARCHAR(255) NOT NULL COMMENT '栏目名称',
    slug VARCHAR(255) UNIQUE NOT NULL COMMENT '栏目别名（URL唯一标识）',
    description TEXT DEFAULT NULL COMMENT '栏目描述',
    template_id INT DEFAULT NULL COMMENT '关联模板ID',
    sort_order INT DEFAULT 0 COMMENT '排序',
    is_visible TINYINT(1) DEFAULT 1 COMMENT '是否可见（1可见, 0隐藏）',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    FOREIGN KEY (parent_id) REFERENCES categories(id) ON DELETE SET NULL,
    FOREIGN KEY (site_id) REFERENCES sites(id) ON DELETE CASCADE,
    FOREIGN KEY (template_id) REFERENCES templates(id) ON DELETE SET NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='栏目表';
```

---

### **2. 文章表（articles）**
用于存储文章，支持栏目关联。

```sql
CREATE TABLE articles (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '文章ID',
    site_id INT NOT NULL COMMENT '所属站点ID',
    category_id INT NOT NULL COMMENT '栏目ID',
    author_id INT NOT NULL COMMENT '作者ID',
    title VARCHAR(255) NOT NULL COMMENT '文章标题',
    slug VARCHAR(255) UNIQUE NOT NULL COMMENT '文章别名（URL唯一标识）',
    summary TEXT COMMENT '摘要',
    content LONGTEXT COMMENT '文章内容',
    cover_image VARCHAR(255) DEFAULT NULL COMMENT '封面图片URL',
    status ENUM('draft', 'published', 'archived') DEFAULT 'draft' COMMENT '文章状态',
    published_at TIMESTAMP NULL DEFAULT NULL COMMENT '发布时间',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    FOREIGN KEY (category_id) REFERENCES categories(id) ON DELETE CASCADE,
    FOREIGN KEY (author_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (site_id) REFERENCES sites(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='文章表';
```

---

### **3. 菜单表（menus）**
支持多级菜单，可用于前台导航。

```sql
CREATE TABLE menus (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '菜单ID',
    site_id INT NOT NULL COMMENT '所属站点ID',
    parent_id INT DEFAULT NULL COMMENT '父级菜单ID',
    name VARCHAR(255) NOT NULL COMMENT '菜单名称',
    slug VARCHAR(255) UNIQUE NOT NULL COMMENT '菜单URL别名',
    link VARCHAR(255) NOT NULL COMMENT '链接地址',
    target ENUM('_self', '_blank') DEFAULT '_self' COMMENT '打开方式',
    sort_order INT DEFAULT 0 COMMENT '排序',
    is_visible TINYINT(1) DEFAULT 1 COMMENT '是否可见（1可见, 0隐藏）',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    FOREIGN KEY (site_id) REFERENCES sites(id) ON DELETE CASCADE,
    FOREIGN KEY (parent_id) REFERENCES menus(id) ON DELETE SET NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='菜单表';
```

---

### **4. 用户表（users）**
包含用户权限管理。

```sql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '用户ID',
    username VARCHAR(100) UNIQUE NOT NULL COMMENT '用户名',
    email VARCHAR(255) UNIQUE NOT NULL COMMENT '邮箱',
    password_hash VARCHAR(255) NOT NULL COMMENT '密码哈希',
    role ENUM('admin', 'editor', 'author', 'subscriber') DEFAULT 'subscriber' COMMENT '用户角色',
    status ENUM('active', 'inactive') DEFAULT 'active' COMMENT '用户状态',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表';
```

---

### **5. 站点配置表（sites）**
支持多站点配置。

```sql
CREATE TABLE sites (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '站点ID',
    name VARCHAR(255) NOT NULL COMMENT '站点名称',
    domain VARCHAR(255) UNIQUE NOT NULL COMMENT '站点域名',
    logo VARCHAR(255) DEFAULT NULL COMMENT '站点Logo',
    settings JSON DEFAULT NULL COMMENT '站点配置（JSON格式）',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='站点配置表';
```

---

### **6. 模板表（templates）**
用于存储模板信息，并关联到栏目。

```sql
CREATE TABLE templates (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '模板ID',
    name VARCHAR(255) NOT NULL COMMENT '模板名称',
    file_path VARCHAR(255) NOT NULL COMMENT '模板文件路径',
    description TEXT DEFAULT NULL COMMENT '模板描述',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='模板表';
```
### **7. 素材表（media_files）**
用于管理站点的素材，如图片、视频、音频、文件等。

```sql
CREATE TABLE media_files (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '素材ID',
    site_id INT NOT NULL COMMENT '所属站点ID',
    user_id INT NOT NULL COMMENT '上传者ID',
    filename VARCHAR(255) NOT NULL COMMENT '文件名',
    file_path VARCHAR(500) NOT NULL COMMENT '文件存储路径',
    file_type ENUM('image', 'video', 'audio', 'document', 'other') NOT NULL COMMENT '文件类型',
    mime_type VARCHAR(100) NOT NULL COMMENT 'MIME类型',
    file_size BIGINT NOT NULL COMMENT '文件大小（字节）',
    width INT DEFAULT NULL COMMENT '图片/视频宽度',
    height INT DEFAULT NULL COMMENT '图片/视频高度',
    duration FLOAT DEFAULT NULL COMMENT '音视频时长（秒）',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '上传时间',
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    FOREIGN KEY (site_id) REFERENCES sites(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE SET NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='素材管理表';
```
### **8. 商品表（products）**
用于存储商品信息，并支持归属 `collections`（商品集合）。

```sql
CREATE TABLE products (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '商品ID',
    site_id INT NOT NULL COMMENT '所属站点ID',
    name VARCHAR(255) NOT NULL COMMENT '商品名称',
    slug VARCHAR(255) UNIQUE NOT NULL COMMENT '商品别名（URL唯一标识）',
    description TEXT DEFAULT NULL COMMENT '商品描述',
    price DECIMAL(10,2) NOT NULL COMMENT '商品价格',
    stock INT DEFAULT 0 COMMENT '库存数量',
    status ENUM('active', 'inactive', 'archived') DEFAULT 'active' COMMENT '商品状态',
    image_url VARCHAR(255) DEFAULT NULL COMMENT '主图片URL',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    FOREIGN KEY (site_id) REFERENCES sites(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='商品表';
```

---

### **9. 商品集合表（collections）**
用于维护商品集合（Collection），可以手动添加商品或使用规则自动匹配商品。

```sql
CREATE TABLE collections (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '集合ID',
    site_id INT NOT NULL COMMENT '所属站点ID',
    name VARCHAR(255) NOT NULL COMMENT '集合名称',
    slug VARCHAR(255) UNIQUE NOT NULL COMMENT '集合别名（URL唯一标识）',
    description TEXT DEFAULT NULL COMMENT '集合描述',
    is_dynamic TINYINT(1) DEFAULT 0 COMMENT '是否为动态集合（0: 手动, 1: 基于规则）',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    FOREIGN KEY (site_id) REFERENCES sites(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='商品集合表';
```

---

### **10. 商品-集合关联表（collection_products）**
用于手动维护 `collections` 和 `products` 的关联关系。

```sql
CREATE TABLE collection_products (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '关联ID',
    collection_id INT NOT NULL COMMENT '集合ID',
    product_id INT NOT NULL COMMENT '商品ID',
    sort_order INT DEFAULT 0 COMMENT '排序',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    FOREIGN KEY (collection_id) REFERENCES collections(id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='商品-集合关联表';
```

---

### **11. 商品列表（product_lists）**
用于管理商品列表，并支持基于规则动态筛选符合条件的商品。

```sql
CREATE TABLE product_lists (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '商品列表ID',
    site_id INT NOT NULL COMMENT '所属站点ID',
    name VARCHAR(255) NOT NULL COMMENT '列表名称',
    slug VARCHAR(255) UNIQUE NOT NULL COMMENT '列表别名（URL唯一标识）',
    description TEXT DEFAULT NULL COMMENT '列表描述',
    is_dynamic TINYINT(1) DEFAULT 1 COMMENT '是否为动态列表（1: 基于规则, 0: 手动）',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    FOREIGN KEY (site_id) REFERENCES sites(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='商品列表表';
```

---

### **12. 商品列表规则表（product_list_rules）**
用于存储 `product_lists` 规则，基于规则动态筛选商品。

```sql
CREATE TABLE product_list_rules (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '规则ID',
    product_list_id INT NOT NULL COMMENT '商品列表ID',
    rule_key VARCHAR(255) NOT NULL COMMENT '规则字段（如 price, stock, status 等）',
    rule_operator ENUM('=', '!=', '>', '>=', '<', '<=', 'LIKE', 'IN') NOT NULL COMMENT '规则操作符',
    rule_value VARCHAR(255) NOT NULL COMMENT '规则值',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    FOREIGN KEY (product_list_id) REFERENCES product_lists(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='商品列表规则表';
```

### **13. 活动表（promotions）**
用于管理活动，包含 **生效时间** 和 **上线状态**。

```sql
CREATE TABLE promotions (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '活动ID',
    site_id INT NOT NULL COMMENT '所属站点ID',
    name VARCHAR(255) NOT NULL COMMENT '活动名称',
    slug VARCHAR(255) UNIQUE NOT NULL COMMENT '活动别名（URL唯一标识）',
    description TEXT DEFAULT NULL COMMENT '活动描述',
    start_time TIMESTAMP NOT NULL COMMENT '活动开始时间',
    end_time TIMESTAMP NOT NULL COMMENT '活动结束时间',
    status ENUM('draft', 'active', 'expired', 'archived') DEFAULT 'draft' COMMENT '活动状态',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    FOREIGN KEY (site_id) REFERENCES sites(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='活动表';
```

---

### **14. 活动-商品关联表（promotion_products）**
用于关联 **活动** 和 **商品**，一个活动可以包含多个商品。

```sql
CREATE TABLE promotion_products (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '关联ID',
    promotion_id INT NOT NULL COMMENT '活动ID',
    product_id INT NOT NULL COMMENT '商品ID',
    discount_type ENUM('fixed', 'percentage') NOT NULL COMMENT '折扣类型（固定金额 / 百分比）',
    discount_value DECIMAL(10,2) NOT NULL COMMENT '折扣值（固定金额或折扣百分比）',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    FOREIGN KEY (promotion_id) REFERENCES promotions(id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='活动-商品关联表';
```

---

### **15. 活动规则表（promotion_rules）**
用于定义活动的适用规则（如最低消费金额、指定分类等）。

```sql
CREATE TABLE promotion_rules (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '规则ID',
    promotion_id INT NOT NULL COMMENT '活动ID',
    rule_key VARCHAR(255) NOT NULL COMMENT '规则字段（如 total_amount, category, user_role 等）',
    rule_operator ENUM('=', '!=', '>', '>=', '<', '<=', 'LIKE', 'IN') NOT NULL COMMENT '规则操作符',
    rule_value VARCHAR(255) NOT NULL COMMENT '规则值',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    FOREIGN KEY (promotion_id) REFERENCES promotions(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='活动规则表';
```

### **16. 会员表（members）**
用于存储会员信息，支持会员等级、成长值、积分等。

```sql
CREATE TABLE members (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '会员ID',
    site_id INT NOT NULL COMMENT '所属站点ID',
    user_id INT NOT NULL COMMENT '用户ID',
    level INT DEFAULT 1 COMMENT '会员等级',
    experience_points INT DEFAULT 0 COMMENT '成长值（用于升级会员等级）',
    total_points INT DEFAULT 0 COMMENT '累计积分',
    available_points INT DEFAULT 0 COMMENT '可用积分',
    status ENUM('active', 'inactive', 'banned') DEFAULT 'active' COMMENT '会员状态',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '注册时间',
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    FOREIGN KEY (site_id) REFERENCES sites(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='会员表';
```

---

### **17. 会员等级表（member_levels）**
用于定义不同等级的会员，以及所需成长值。

```sql
CREATE TABLE member_levels (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '等级ID',
    site_id INT NOT NULL COMMENT '所属站点ID',
    level INT UNIQUE NOT NULL COMMENT '会员等级',
    name VARCHAR(255) NOT NULL COMMENT '等级名称',
    required_experience INT NOT NULL COMMENT '所需成长值',
    discount_percentage DECIMAL(5,2) DEFAULT 0 COMMENT '会员折扣（百分比）',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    FOREIGN KEY (site_id) REFERENCES sites(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='会员等级表';
```

---

### **18. 积分表（points_transactions）**
记录会员积分的变动（增加、使用、过期等）。

```sql
CREATE TABLE points_transactions (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '交易ID',
    member_id INT NOT NULL COMMENT '会员ID',
    transaction_type ENUM('earn', 'redeem', 'expire', 'adjust') NOT NULL COMMENT '交易类型',
    points INT NOT NULL COMMENT '积分变动数量（正数代表增加，负数代表减少）',
    description VARCHAR(255) DEFAULT NULL COMMENT '交易描述',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '交易时间',
    FOREIGN KEY (member_id) REFERENCES members(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='积分交易表';
```

---

### **19. 会员权益表（member_benefits）**
用于存储不同等级会员可享受的权益。

```sql
CREATE TABLE member_benefits (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '权益ID',
    level INT NOT NULL COMMENT '会员等级',
    benefit_name VARCHAR(255) NOT NULL COMMENT '权益名称',
    benefit_description TEXT DEFAULT NULL COMMENT '权益描述',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    FOREIGN KEY (level) REFERENCES member_levels(level) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='会员权益表';
```
### **20. 库存表（inventory）**
用于存储商品的库存信息，支持多仓库管理、库存变动记录等。

```sql
CREATE TABLE inventory (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '库存ID',
    site_id INT NOT NULL COMMENT '所属站点ID',
    product_id INT NOT NULL COMMENT '商品ID',
    warehouse_id INT NOT NULL COMMENT '仓库ID',
    stock INT NOT NULL COMMENT '当前库存数量',
    reserved_stock INT DEFAULT 0 COMMENT '已预留库存（未支付订单）',
    min_stock_threshold INT DEFAULT 0 COMMENT '库存预警阈值',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    FOREIGN KEY (site_id) REFERENCES sites(id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE,
    FOREIGN KEY (warehouse_id) REFERENCES warehouses(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='库存表';
```

---

### **21. 仓库表（warehouses）**
用于存储仓库信息，支持多仓库管理。

```sql
CREATE TABLE warehouses (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '仓库ID',
    site_id INT NOT NULL COMMENT '所属站点ID',
    name VARCHAR(255) NOT NULL COMMENT '仓库名称',
    location VARCHAR(255) NOT NULL COMMENT '仓库位置',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    FOREIGN KEY (site_id) REFERENCES sites(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='仓库表';
```

---

### **22. 库存变动记录表（inventory_transactions）**
用于记录库存变动，如 **入库、出库、销售、退货** 等操作。

```sql
CREATE TABLE inventory_transactions (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '交易ID',
    inventory_id INT NOT NULL COMMENT '库存ID',
    transaction_type ENUM('restock', 'sale', 'return', 'adjustment') NOT NULL COMMENT '变动类型',
    quantity INT NOT NULL COMMENT '变动数量（正数表示增加，负数表示减少）',
    description VARCHAR(255) DEFAULT NULL COMMENT '变动描述',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '变动时间',
    FOREIGN KEY (inventory_id) REFERENCES inventory(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='库存变动记录表';
```
