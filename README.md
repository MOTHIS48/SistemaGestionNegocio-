// main.dart
import 'package:flutter/material.dart';

void main() {
  runApp(SistemaGestionApp());
}

class SistemaGestionApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Sistema Gestión Negocio',
      theme: ThemeData(primarySwatch: Colors.teal),
      home: LoginPage(),
    );
  }
}

// --- Login sin contraseña ---
class LoginPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: ElevatedButton(
          child: Text('Entrar al Sistema'),
          onPressed: () {
            Navigator.pushReplacement(
                context, MaterialPageRoute(builder: (_) => Dashboard()));
          },
        ),
      ),
    );
  }
}

// --- Dashboard ---
class Dashboard extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Panel Principal')),
      body: ListView(
        padding: EdgeInsets.all(20),
        children: [
          _buildTile(context, 'Registro de Ventas', RegistroVentas()),
          _buildTile(context, 'Inventario', InventarioPage()),
          _buildTile(context, 'Gastos e Ingresos', GastosPage()),
          _buildTile(context, 'Reportes', ReportesPage()),
          _buildTile(context, 'Configuración del Negocio', ConfiguracionPage()),
        ],
      ),
    );
  }

  Widget _buildTile(BuildContext context, String title, Widget page) {
    return Card(
      child: ListTile(
        title: Text(title),
        trailing: Icon(Icons.arrow_forward_ios),
        onTap: () {
          Navigator.push(context, MaterialPageRoute(builder: (_) => page));
        },
      ),
    );
  }
}

// --- Registro de Ventas ---
class RegistroVentas extends StatefulWidget {
  @override
  _RegistroVentasState createState() => _RegistroVentasState();
}

class _RegistroVentasState extends State<RegistroVentas> {
  Map<String, int> inventario = {
    'Producto A': 50,
    'Producto B': 30,
    'Producto C': 20,
  };

  Map<String, int> carrito = {};

  String? productoSeleccionado;
  int cantidad = 1;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Registro de Ventas')),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            DropdownButton<String>(
              hint: Text('Seleccionar Producto'),
              value: productoSeleccionado,
              items: inventario.keys
                  .map((p) => DropdownMenuItem(
                        child: Text('$p (Stock: ${inventario[p]})'),
                        value: p,
                      ))
                  .toList(),
              onChanged: (val) {
                setState(() {
                  productoSeleccionado = val;
                  cantidad = 1;
                });
              },
            ),
            SizedBox(height: 10),
            Row(
              children: [
                Text('Cantidad:'),
                SizedBox(width: 10),
                IconButton(
                    icon: Icon(Icons.remove),
                    onPressed: () {
                      if (cantidad > 1) {
                        setState(() {
                          cantidad--;
                        });
                      }
                    }),
                Text('$cantidad'),
                IconButton(
                    icon: Icon(Icons.add),
                    onPressed: () {
                      if (productoSeleccionado != null &&
                          cantidad < inventario[productoSeleccionado]!) {
                        setState(() {
                          cantidad++;
                        });
                      }
                    }),
              ],
            ),
            ElevatedButton(
                onPressed: productoSeleccionado == null ? null : () {
                  if (inventario[productoSeleccionado]! >= cantidad) {
                    setState(() {
                      inventario[productoSeleccionado!] =
                          inventario[productoSeleccionado!]! - cantidad;
                      carrito[productoSeleccionado!] =
                          (carrito[productoSeleccionado!] ?? 0) + cantidad;
                      productoSeleccionado = null;
                      cantidad = 1;
                    });
                    ScaffoldMessenger.of(context).showSnackBar(
                        SnackBar(content: Text('Venta agregada')));
                  } else {
                    ScaffoldMessenger.of(context).showSnackBar(
                        SnackBar(content: Text('Stock insuficiente')));
                  }
                },
                child: Text('Agregar Venta')),
            Divider(),
            Text('Ventas registradas:', style: TextStyle(fontWeight: FontWeight.bold)),
            Expanded(
              child: ListView(
                children: carrito.entries
                    .map((e) => ListTile(
                          title: Text('${e.key}'),
                          trailing: Text('Cantidad: ${e.value}'),
                        ))
                    .toList(),
              ),
            )
          ],
        ),
      ),
    );
  }
}

// --- Inventario ---
class InventarioPage extends StatefulWidget {
  @override
  _InventarioPageState createState() => _InventarioPageState();
}

class _InventarioPageState extends State<InventarioPage> {
  Map<String, int> inventario = {
    'Producto A': 50,
    'Producto B': 30,
    'Producto C': 20,
  };

  final _nombreController = TextEditingController();
  final _cantidadController = TextEditingController();

  @override
  void dispose() {
    _nombreController.dispose();
    _cantidadController.dispose();
    super.dispose();
  }

  void _agregarProducto() {
    final nombre = _nombreController.text.trim();
    final cantidad = int.tryParse(_cantidadController.text.trim());

    if (nombre.isEmpty || cantidad == null || cantidad <= 0) {
      ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('Ingrese datos válidos')));
      return;
    }

    setState(() {
      inventario[nombre] = (inventario[nombre] ?? 0) + cantidad;
    });

    _nombreController.clear();
    _cantidadController.clear();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Inventario')),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            TextField(
              controller: _nombreController,
              decoration: InputDecoration(labelText: 'Nombre del Producto'),
            ),
            TextField(
              controller: _cantidadController,
              decoration: InputDecoration(labelText: 'Cantidad'),
              keyboardType: TextInputType.number,
            ),
            ElevatedButton(onPressed: _agregarProducto, child: Text('Agregar Producto')),
            Divider(),
            Expanded(
              child: ListView(
                children: inventario.entries
                    .map((e) => ListTile(
                          title: Text(e.key),
                          trailing: Text('Stock: ${e.value}'),
                        ))
                    .toList(),
              ),
            )
          ],
        ),
      ),
    );
  }
}

// --- Gastos e Ingresos ---
class GastosPage extends StatefulWidget {
  @override
  _GastosPageState createState() => _GastosPageState();
}

class _GastosPageState extends State<GastosPage> {
  List<Map<String, dynamic>> registros = [];

  final _descripcionController = TextEditingController();
  final _montoController = TextEditingController();
  String tipo = 'Gasto';
  String categoria = 'General';

  final categorias = ['General', 'Marketing', 'Operación', 'Otros'];

  void _agregarRegistro() {
    final descripcion = _descripcionController.text.trim();
    final monto = double.tryParse(_montoController.text.trim());

    if (descripcion.isEmpty || monto == null || monto <= 0) {
      ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('Ingrese datos válidos')));
      return;
    }

    setState(() {
      registros.add({
        'descripcion': descripcion,
        'monto': monto,
        'tipo': tipo,
        'categoria': categoria,
        'fecha': DateTime.now(),
      });
    });

    _descripcionController.clear();
    _montoController.clear();
  }

  @override
  void dispose() {
    _descripcionController.dispose();
    _montoController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Gastos e Ingresos')),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            DropdownButton<String>(
              value: tipo,
              items: ['Gasto', 'Ingreso']
                  .map((e) => DropdownMenuItem(child: Text(e), value: e))
                  .toList(),
              onChanged: (val) {
                if (val != null) setState(() => tipo = val);
              },
            ),
            DropdownButton<String>(
              value: categoria,
              items: categorias
                  .map((e) => DropdownMenuItem(child: Text(e), value: e))
                  .toList(),
              onChanged: (val) {
                if (val != null) setState(() => categoria = val);
              },
            ),
            TextField(
              controller: _descripcionController,
              decoration: InputDecoration(labelText: 'Descripción'),
            ),
            TextField(
              controller: _montoController,
              decoration: InputDecoration(labelText: 'Monto'),
              keyboardType: TextInputType.numberWithOptions(decimal: true),
            ),
            ElevatedButton(onPressed: _agregarRegistro, child: Text('Agregar')),
            Divider(),
            Expanded(
              child: ListView(
                children: registros.reversed
                    .map((r) => ListTile(
                          title: Text('${r['descripcion']} - ${r['categoria']}'),
                          subtitle: Text('${r['fecha']}'),
                          trailing: Text(
                            '${r['tipo'] == 'Gasto' ? '-' : '+'}\$${r['monto'].toStringAsFixed(2)}',
                            style: TextStyle(
                                color: r['tipo'] == 'Gasto' ? Colors.red : Colors.green,
                                fontWeight: FontWeight.bold),
                          ),
                        ))
                    .toList(),
              ),
            )
          ],
        ),
