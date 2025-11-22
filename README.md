# n8n Automation Santex Course Summary 🚀

## Introduction to n8n ⚙️

n8n is an automation platform that allows you to create workflows connecting apps, APIs, and services with low code.

---

## Why Use n8n ❓

- Saves time by automating repetitive tasks  
- Connects multiple tools  
- Visual builder  
- Cloud or self-hosted  
- Allows API calls and JavaScript actions  

---

## Core Features ⭐

- Drag-and-drop workflow builder  
- Hundreds of integrations  
- Conditional logic  
- Loops and transformations  
- Webhooks  
- Error handling  
- Version control  

---

## Common Use Cases 📚

- Lead automation  
- Notifications and reminders  
- API data processing  
- Syncing platforms  
- Automated reports  
- Real-time monitoring  

---

## Workflow Structure 🔧

### Basic Workflow Example

1. Trigger receives data  
2. Validation  
3. Processing  
4. Notification  
5. Logging  

---

## Function Node Code Example 🧩

```javascript
return [
  {
    name: $json.name.trim(),
    email: $json.email.toLowerCase(),
    timestamp: new Date().toISOString()
  }
];
```

---

## IF Node Condition Example 🔍

```json
{
  "rules": [
    {
      "operation": "isEmpty",
      "input": "email"
    }
  ]
}
```

---

## HTTP Request Example (POST) 🌐

```json
{
  "method": "POST",
  "url": "https://api.example.com/submit",
  "body": {
    "name": "={{$json.name}}",
    "email": "={{$json.email}}"
  }
}
```

---

## Best Practices 📝

- Name nodes clearly  
- Use modular workflows  
- Add error handling  
- Use environment variables  
- Document workflows  

---

## Workflow Backup Tips 💾

- Export JSON files  
- Use versioning  
- Sync workflows  

---

# AI Agent Output Parser Node 🤖🧹

This node cleans the output from an AI agent, removes backticks, parses JSON safely, and formats the response for the rest of the workflow.

```jsx
const rawOutput = $json.output || "";
const cleaned = rawOutput.replace(/```json|```/g, "").trim();

let parsed;
try {
  parsed = JSON.parse(cleaned);
} catch (err) {
  console.error("❌ Error parseando JSON:", err.message);
  console.error("Output:", cleaned.substring(0, 200));
  parsed = {
    output: rawOutput || "Disculpá, hubo un error. Escribí 'categorías'.",
    admin_notice: ""
  };
}

const output = (parsed.output?.trim?.() || "");
const admin_notice = (parsed.admin_notice?.trim?.() || "");
const hasAdminNotice = !!(admin_notice && admin_notice.length > 0);

if (!output) {
  console.error("⚠️ Output vacío");
  return [{
    output: "Disculpá, no entendí. ¿Querés ver las categorías?",
    hasAdminNotice: false
  }];
}

if (hasAdminNotice) {
  console.log("📢 Notificación admin:", admin_notice.substring(0, 100));
}

const result = { output, hasAdminNotice };
if (hasAdminNotice) result.admin_notice = admin_notice;

return [result];
```

---

# Google Sheets Helper Scripts 📊🔧

## JSON Parser for Sheet Requests 🧩

```jsx
// Obtener datos del workflow que nos llamó
const inputData = $input.first().json;

console.log('📥 Input recibido:', JSON.stringify(inputData));

// Los datos vienen en query como string JSON, necesitamos parsearlo
let operation = '';
let params = {};

if (inputData.query) {
  try {
    // Parsear el string JSON
    const parsed = JSON.parse(inputData.query);
    operation = parsed.operation || '';
    params = parsed.params || {};
  } catch (err) {
    console.error('❌ Error parseando query:', err.message);
    throw new Error('Query no es JSON válido');
  }
} else if (inputData.operation) {
  // Fallback: si vienen directamente
  operation = inputData.operation;
  params = inputData.params || {};
}

console.log(`🔧 Operación solicitada: ${operation}`);
console.log(`📋 Parámetros:`, JSON.stringify(params));

if (!operation) {
  throw new Error('No se especificó operation');
}

return [{
  operation,
  params,
  requestId: Date.now()
}];
```

---

## Catalog Processor 🗂️

```jsx
const rows = $input.all();

if (rows.length === 0) {
  return [{ success: false, catalog: [], categories: [], error: 'No se encontraron datos' }];
}

const catalog = rows.map(row => row.json);
const categories = [...new Set(catalog.map(item => item.Categoria))].filter(c => c).sort();

console.log(`✅ Catálogo: ${catalog.length} productos, ${categories.length} categorías`);

return [{
  success: true,
  catalog,
  categories,
  total_products: catalog.length
}];
```

---

## Category Filter 🔎

```jsx
const allData = $input.all();
const category = $('Parse Request').item.json.params.category;

const products = allData
  .map(row => row.json)
  .filter(item => item.Categoria?.toLowerCase() === category?.toLowerCase())
  .sort((a, b) => a.Producto.localeCompare(b.Producto));

console.log(`📦 ${category}: ${products.length} productos`);

return [{
  success: true,
  category,
  products,
  total: products.length
}];
```

---

## Stock Validator 🏷️📦

```jsx
// 🎓 FUNCIÓN: Validar Stock Sin Actualizar
// Propósito: Verificar disponibilidad antes de confirmar al cliente

const orderItems = $('Parse Request').item.json.params.order_items || [];
const allData = $input.all().map(row => row.json);

console.log('🔍 VALIDACIÓN: Verificando stock para:', orderItems);

// Validación básica
if (orderItems.length === 0) {
  return [{ 
    success: false, 
    error: 'carrito_vacio',
    mensaje: 'El carrito está vacío'
  }];
}

const stockIssues = [];    // Problemas encontrados
const validItems = [];     // Items que SÍ tienen stock

// Revisar cada producto del pedido
for (const item of orderItems) {
  
  // 1️⃣ Limpiar nombre del producto (remover marca entre paréntesis)
  let productName = item.producto || '';
  const match = productName.match(/^([^(]+)/);
  if (match) {
    productName = match[1].trim();
  }
  
  console.log(`🔍 Buscando: "${item.producto}" → Limpio: "${productName}"`);
  
  // 2️⃣ Buscar el producto en el catálogo
  const product = allData.find(p => 
    p.Producto?.toLowerCase().trim() === productName.toLowerCase().trim()
  );
  
  // 3️⃣ Verificar si existe el producto
  if (!product) {
    stockIssues.push({
      producto: item.producto,
      tipo: 'no_encontrado',
      mensaje: `"${item.producto}" no existe en nuestro catálogo`,
      sugerencia: 'Verifica el nombre o consulta las categorías disponibles'
    });
    continue;
  }
  
  // 4️⃣ Verificar stock disponible
  const currentStock = parseInt(product.Stock) || 0;
  
  if (currentStock === 0) {
    stockIssues.push({
      producto: product.Producto,
      tipo: 'agotado',
      mensaje: `"${product.Producto}" está agotado`,
      stockDisponible: 0,
      cantidadSolicitada: item.cantidad,
      sugerencia: 'Prueba con otro producto similar'
    });
    
  } else if (currentStock < item.cantidad) {
    stockIssues.push({
      producto: product.Producto,
      tipo: 'stock_insuficiente',
      mensaje: `"${product.Producto}" solo tiene ${currentStock} unidades disponibles`,
      stockDisponible: currentStock,
      cantidadSolicitada: item.cantidad,
      sugerencia: `¿Quieres llevar ${currentStock} unidades en lugar de ${item.cantidad}?`
    });
    
  } else {
    validItems.push({
      producto: product.Producto,
      cantidad: item.cantidad,
      stockActual: currentStock,
      precio: product.Precio,
      ok: true
    });
  }
}

const hasIssues = stockIssues.length > 0;

console.log(`✅ Validación completa: ${validItems.length} OK, ${stockIssues.length} problemas`);

return [{
  success: !hasIssues,
  hasStockIssues: hasIssues,
  stockIssues,
  validItems,
  canProceed: !hasIssues,
  totalValidItems: validItems.length,
  totalIssues: stockIssues.length
}];
```

---

## Order Processor 🛒⚙️

```jsx
console.log('🔍 DEBUG Process Order');

const orderItems = $('Parse Request').item.json.params.order_items || [];
const allData = $input.all().map(row => row.json);

console.log('🛒 Order items recibidos:', JSON.stringify(orderItems));
console.log('📦 Total productos en catálogo:', allData.length);

if (orderItems.length === 0) {
  return [{ success: false, error: 'No hay items en el pedido' }];
}

const results = [];
const lowStockAlerts = [];
const updates = [];

for (const item of orderItems) {
  let productName = item.producto || '';
  
  const match = productName.match(/^([^(]+)/);
  if (match) {
    productName = match[1].trim();
  }
  
  console.log(`🔍 Buscando: "${item.producto}" → Limpio: "${productName}"`);
  
  const productIndex = allData.findIndex(p => 
    p.Producto?.toLowerCase().trim() === productName.toLowerCase().trim()
  );
  
  if (productIndex === -1) {
    console.error(`❌ No encontrado: "${productName}"`);
    results.push({ 
      producto: item.producto, 
      success: false, 
      error: 'No encontrado' 
    });
    continue;
  }
  
  const product = allData[productIndex];
  const currentStock = parseInt(product.Stock) || 0;
  const newStock = Math.max(0, currentStock - item.cantidad);
  
  console.log(`✅ Encontrado: ${product.Producto} - Stock: ${currentStock} → ${newStock}`);
  
  updates.push({
    row: productIndex + 2,
    producto: product.Producto,
    stockAnterior: currentStock,
    newStock
  });
  
  if (newStock < 5) {
    lowStockAlerts.push({ producto: product.Producto, stock: newStock });
  }
  
  results.push({
    producto: product.Producto,
    cantidad: item.cantidad,
    stockAnterior: currentStock,
    stockNuevo: newStock,
    success: true
  });
}

console.log(`🛒 Procesado: ${results.filter(r => r.success).length}/${results.length}`);
console.log(`📝 Updates a aplicar: ${updates.length}`);

return [{
  success: true,
  results,
  lowStockAlerts,
  updates,
  totalProcessed: results.filter(r => r.success).length
}];
```

---

## Update Preparation Node 🧾➡️📊

```jsx
const processResult = $json;
const updates = processResult.updates || [];

if (updates.length === 0) {
  return [];
}

return updates.map(update => ({
  row: update.row,
  producto: update.producto,
  newStock: update.newStock
}));
```

---

# Links to Sections Above 🔗

- [Introduction](#introduction-to-n8n-️)  
- [AI Agent Parser](#ai-agent-output-parser-node-)  
- [Google Sheets Helpers](#google-sheets-helper-scripts-)  
- [Stock Validator](#stock-validator-)  
- [Order Processor](#order-processor-)  
