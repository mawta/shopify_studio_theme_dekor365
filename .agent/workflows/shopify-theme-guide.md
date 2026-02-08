---
description: Hướng dẫn chi tiết thêm/sửa/xóa Shopify theme cho các AI model
---

# Kịch Bản Phát Triển Shopify Theme

> [!IMPORTANT]
> Workflow này dành cho các AI model để thực hiện các tác vụ trên Shopify theme một cách chuẩn xác.

---

## 1. CẤU TRÚC THƯ MỤC SHOPIFY THEME

```
theme/
├── assets/          # CSS, JS, images, fonts
├── config/          # Cấu hình theme (settings_schema.json, settings_data.json)
├── layout/          # Layout chính (theme.liquid)
├── locales/         # Đa ngôn ngữ (.json)
├── sections/        # Các section reusable
├── snippets/        # Code snippets nhỏ
└── templates/       # Templates cho từng loại page
```

### Mô tả từng thư mục:

| Thư mục | Mục đích | Format |
|---------|----------|--------|
| `assets/` | Chứa CSS, JS, images | `.css`, `.js`, `.png`, `.svg` |
| `config/` | Cấu hình theme settings | `.json` |
| `layout/` | Template wrapper cho toàn bộ trang | `.liquid` |
| `locales/` | Text đa ngôn ngữ | `.json` |
| `sections/` | Components có thể cấu hình từ admin | `.liquid` |
| `snippets/` | Code tái sử dụng nhỏ | `.liquid` |
| `templates/` | Templates cho product, collection, page... | `.json` hoặc `.liquid` |

---

## 2. THÊM SECTION MỚI

### Bước 1: Tạo file section

**Vị trí:** `sections/[ten-section].liquid`

**Template chuẩn:**

```liquid
{% comment %}
  Section: [Tên Section]
  Mô tả: [Mô tả chức năng]
{% endcomment %}

<div class="section-{{ section.id }} {{ section.settings.custom_class }}">
  <div class="container">
    {% if section.settings.heading != blank %}
      <h2 class="section-heading">{{ section.settings.heading }}</h2>
    {% endif %}

    {%- for block in section.blocks -%}
      {% case block.type %}
        {% when 'item' %}
          <div class="block-item" {{ block.shopify_attributes }}>
            {{ block.settings.content }}
          </div>
      {% endcase %}
    {%- endfor -%}
  </div>
</div>

{% schema %}
{
  "name": "Tên Section",
  "tag": "section",
  "class": "section-class",
  "settings": [
    {
      "type": "text",
      "id": "heading",
      "label": "Tiêu đề",
      "default": "Tiêu đề mặc định"
    },
    {
      "type": "text",
      "id": "custom_class",
      "label": "Custom CSS Class"
    }
  ],
  "blocks": [
    {
      "type": "item",
      "name": "Item",
      "settings": [
        {
          "type": "richtext",
          "id": "content",
          "label": "Nội dung"
        }
      ]
    }
  ],
  "presets": [
    {
      "name": "Tên Section"
    }
  ]
}
{% endschema %}
```

### Bước 2: Thêm CSS (nếu cần)

**Vị trí:** `assets/section-[ten-section].css`

Tham chiếu trong section:
```liquid
{{ 'section-ten-section.css' | asset_url | stylesheet_tag }}
```

---

## 3. SỬA SECTION CÓ SẴN

### Quy trình:

1. **Tìm file section:** `sections/[ten-section].liquid`
2. **Đọc hiểu cấu trúc:** Xem phần HTML/Liquid và `{% schema %}`
3. **Chỉnh sửa HTML/Liquid** phía trên `{% schema %}`
4. **Chỉnh sửa settings** trong `{% schema %}` nếu cần thêm options
5. **Test:** Đảm bảo không có syntax error

### Lưu ý quan trọng:
- KHÔNG thay đổi `id` của settings đã có (sẽ mất data từ admin)
- CÓ THỂ thêm settings mới
- Giữ nguyên cấu trúc JSON trong schema

---

## 4. THÊM SNIPPET

### Vị trí: `snippets/[ten-snippet].liquid`

**Template:**
```liquid
{% comment %}
  Snippet: [Tên]
  Sử dụng: {% render 'ten-snippet', param1: value1 %}
{% endcomment %}

{% comment %} Khai báo tham số {% endcomment %}
{%- liquid
  assign param1 = param1 | default: 'default_value'
-%}

<div class="snippet-wrapper">
  {{ param1 }}
</div>
```

### Cách sử dụng snippet:
```liquid
{% render 'ten-snippet', param1: 'value1', param2: 'value2' %}
```

---

## 5. CHỈNH SỬA TEMPLATE

### Templates dạng JSON (mới):

**Vị trí:** `templates/[type].json`

Ví dụ `templates/product.json`:
```json
{
  "sections": {
    "main": {
      "type": "main-product",
      "settings": {
        "show_vendor": true
      }
    },
    "related": {
      "type": "related-products",
      "settings": {}
    }
  },
  "order": ["main", "related"]
}
```

### Templates dạng Liquid (cũ):

**Vị trí:** `templates/[type].liquid`

```liquid
{% section 'header' %}
{% section 'main-product' %}
{% section 'footer' %}
```

---

## 6. THÊM SETTINGS VÀO SCHEMA

### Vị trí: `config/settings_schema.json`

**Cấu trúc:**
```json
[
  {
    "name": "theme_info",
    "theme_name": "Theme Name",
    "theme_version": "1.0.0"
  },
  {
    "name": "Tên nhóm settings",
    "settings": [
      {
        "type": "text",
        "id": "setting_id",
        "label": "Label hiển thị",
        "default": "Giá trị mặc định"
      }
    ]
  }
]
```

### Các loại input phổ biến:

| Type | Mô tả |
|------|-------|
| `text` | Input text đơn |
| `textarea` | Textarea nhiều dòng |
| `richtext` | Editor với formatting |
| `image_picker` | Chọn ảnh |
| `url` | Link |
| `checkbox` | Boolean |
| `range` | Slider số |
| `select` | Dropdown |
| `color` | Color picker |
| `font_picker` | Chọn font |

---

## 7. LIQUID SYNTAX CHEATSHEET

### Biến và Output:
```liquid
{{ product.title }}                    {# In giá trị #}
{{ product.title | upcase }}           {# Sử dụng filter #}
{{ product.price | money }}            {# Format tiền #}
```

### Điều kiện:
```liquid
{% if condition %}
  ...
{% elsif other_condition %}
  ...
{% else %}
  ...
{% endif %}

{% unless condition %}
  ...
{% endunless %}
```

### Vòng lặp:
```liquid
{% for item in collection %}
  {{ item.title }}
  {% if forloop.first %}Đầu tiên{% endif %}
  {% if forloop.last %}Cuối cùng{% endif %}
  {{ forloop.index }}      {# 1, 2, 3... #}
  {{ forloop.index0 }}     {# 0, 1, 2... #}
{% endfor %}
```

### Gán biến:
```liquid
{% assign my_var = 'value' %}
{% assign products = collection.products | where: 'available', true %}

{%- liquid
  assign var1 = 'value1'
  assign var2 = 'value2'
-%}
```

### Capture (build string):
```liquid
{% capture my_string %}
  Nội dung với {{ product.title }}
{% endcapture %}
```

### Render snippets:
```liquid
{% render 'snippet-name' %}
{% render 'snippet-name', product: product, show_title: true %}
```

### Include sections:
```liquid
{% section 'section-name' %}
```

---

## 8. CHECKLIST TRƯỚC KHI HOÀN THÀNH

- [ ] Syntax Liquid đúng (không có lỗi `{% %}` `{{ }}`)
- [ ] JSON schema hợp lệ (sử dụng tool validate JSON)
- [ ] Đã khai báo biến trước khi sử dụng
- [ ] Không có hardcode text (sử dụng settings hoặc locales)
- [ ] CSS class không conflict với các section khác
- [ ] Đã test responsive (nếu có CSS)

---

## 9. FILES QUAN TRỌNG CẦN XEM

Khi bắt đầu làm việc với theme, luôn đọc các file này trước:

1. `layout/theme.liquid` - Layout chính
2. `config/settings_schema.json` - Toàn bộ settings
3. `assets/base.css` hoặc `assets/global.css` - CSS nền tảng
4. `snippets/` - Các snippets có sẵn để tái sử dụng

---

## 10. LƯU Ý ĐẶC BIỆT CHO AI MODEL

> [!CAUTION]
> - **KHÔNG XÓA** settings có `id` đã tồn tại → mất data người dùng đã cấu hình
> - **KHÔNG THAY ĐỔI** tên file section/snippet đang được sử dụng
> - **LUÔN BACKUP** nội dung file trước khi sửa lớn

> [!TIP]
> - Đọc file hiện tại TRƯỚC khi sửa
> - Hỏi user nếu không chắc chắn về yêu cầu
> - Sử dụng comment tiếng Việt để giải thích code
