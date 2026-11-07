import 'package:flutter/material.dart';
import 'package:flutter_map/flutter_map.dart';
import 'package:latlong2/latlong.dart';

void main() {
  runApp(const MikrotikApp());
}

class MikrotikApp extends StatelessWidget {
  const MikrotikApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'مراقب الميكروتيك',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        brightness: Brightness.dark,
        primarySwatch: Colors.emerald,
        scaffoldBackgroundColor: const Color(0xFF0F172A),
      ),
      home: const DashboardScreen(),
    );
  }
}

class ModemModel {
  final String id;
  final String name;
  final LatLng location;
  bool isOnline;
  final String ip;

  ModemModel({
    required this.id,
    required this.name,
    required this.location,
    required this.isOnline,
    required this.ip,
  });
}

class DashboardScreen extends StatefulWidget {
  const DashboardScreen({super.key});

  @override
  State<DashboardScreen> createState() => _DashboardScreenState();
}

class _DashboardScreenState extends State<DashboardScreen> {
  int _currentIndex = 0;
  final MapController _mapController = MapController();

  // قائمة المودمات التجريبية
  final List<ModemModel> _modems = [
    ModemModel(id: 'MOD-101', name: 'مودم السوق المركزي', location: const LatLng(15.3694, 44.1910), isOnline: true, ip: '192.168.88.10'),
    ModemModel(id: 'MOD-102', name: 'مودم الجولة الشمالية', location: const LatLng(15.3750, 44.1850), isOnline: true, ip: '192.168.88.11'),
    ModemModel(id: 'MOD-103', name: 'مودم شارع الستين', location: const LatLng(15.3620, 44.1950), isOnline: false, ip: '192.168.88.12'),
    ModemModel(id: 'MOD-104', name: 'مودم الحي الغربي', location: const LatLng(15.3580, 44.1800), isOnline: true, ip: '192.168.88.13'),
  ];

  void _flyToModem(LatLng location) {
    setState(() {
      _currentIndex = 0; // الانتقال لتبويب الخريطة
    });
    _mapController.move(location, 16.0);
  }

  void _toggleSimulation() {
    setState(() {
      // عكس حالة المودم رقم 103 للاختبار
      var modem = _modems.firstWhere((m) => m.id == 'MOD-103');
      modem.isOnline = !modem.isOnline;
    });
  }

  @override
  Widget build(BuildContext context) {
    int offlineCount = _modems.where((m) => !m.isOnline).length;
    int onlineCount = _modems.where((m) => m.isOnline).length;

    final List<Widget> screens = [
      // تبويب الخريطة
      Stack(
        children: [
          FlutterMap(
            mapController: _mapController,
            options: const MapOptions(
              initialCenter: LatLng(15.3694, 44.1910),
              initialZoom: 13.0,
            ),
            children: [
              TileLayer(
                urlTemplate: 'https://tile.openstreetmap.org/{z}/{x}/{y}.png',
                userAgentPackageName: 'com.example.mikrotik_monitor',
              ),
              MarkerLayer(
                markers: _modems.map((modem) {
                  return Marker(
                    point: modem.location,
                    width: 40,
                    height: 40,
                    child: GestureDetector(
                      onTap: () {
                        showModalBottomSheet(
                          context: context,
                          backgroundColor: const Color(0xFF1E293B),
                          builder: (context) => Container(
                            padding: const EdgeInsets.all(20),
                            child: Column(
                              mainAxisSize: MainAxisSize.min,
                              crossAxisAlignment: CrossAxisAlignment.start,
                              children: [
                                Text(modem.name, style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
                                const SizedBox(height: 8),
                                Text('رقم المودم: ${modem.id}'),
                                Text('عنوان الـ IP: ${modem.ip}'),
                                Text('الحالة: ${modem.isOnline ? "متصل" : "منقطع"}', 
                                  style: TextStyle(color: modem.isOnline ? Colors.green : Colors.red, fontWeight: FontWeight.bold)),
                              ],
                            ),
                          ),
                        );
                      },
                      child: Container(
                        decoration: BoxDecoration(
                          color: modem.isOnline ? Colors.green : Colors.red,
                          shape: BoxShape.circle,
                          border: Border.all(color: Colors.white, width: 2),
                          boxShadow: [
                            BoxShadow(
                              color: (modem.isOnline ? Colors.green : Colors.red).withOpacity(0.6),
                              blurRadius: 8,
                              spreadRadius: 2,
                            )
                          ],
                        ),
                        child: Icon(
                          modem.isOnline ? Icons.router : Icons.error_outline,
                          color: Colors.white,
                          size: 20,
                        ),
                      ),
                    ),
                  );
                }).toList(),
              ),
            ],
          ),
          // زر محاكاة الانقطاع
          Positioned(
            top: 16,
            right: 16,
            child: FloatingActionButton.extended(
              onPressed: _toggleSimulation,
              backgroundColor: Colors.amber[800],
              icon: const Icon(Icons.bolt, color: Colors.white),
              label: const Text('محاكاة انقطاع مودم', style: TextStyle(color: Colors.white)),
            ),
          )
        ],
      ),

      // تبويب قائمة الأجهزة المنقطعة
      ListView(
        padding: const EdgeInsets.all(16),
        children: [
          const Text('التنبيهات والأجهزة المنقطعة', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
          const SizedBox(height: 12),
          if (offlineCount == 0)
            const Center(child: Padding(padding: EdgeInsets.all(40), child: Text('جميع المودمات متصلة بشكل سليم 👍', style: TextStyle(color: Colors.green))))
          else
            ..._modems.where((m) => !m.isOnline).map((modem) {
              return Card(
                color: Colors.red.withOpacity(0.15),
                shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12), side: const BorderSide(color: Colors.red, width: 1)),
                margin: const EdgeInsets.only(bottom: 12),
                child: ListTile(
                  leading: const Icon(Icons.warning_amber_rounded, color: Colors.red, size: 36),
                  title: Text(modem.name, style: const TextStyle(fontWeight: FontWeight.bold)),
                  subtitle: Text('الرقم: ${modem.id} | IP: ${modem.ip}'),
                  trailing: ElevatedButton(
                    onPressed: () => _flyToModem(modem.location),
                    style: ElevatedButton.styleFrom(backgroundColor: Colors.red),
                    child: const Text('تحديد الموقع 📍', style: TextStyle(color: Colors.white)),
                  ),
                ),
              );
            }),
        ],
      ),

      // تبويب إحصائيات النظام
      Padding(
        padding: const EdgeInsets.all(20),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text('معلومات الشبكة والحماية', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
            const SizedBox(height: 20),
            _buildStatCard('إجمالي المودمات', '${_modems.length}', Icons.devices, Colors.blue),
            _buildStatCard('المودمات المتصلة', '$onlineCount', Icons.check_circle, Colors.green),
            _buildStatCard('المودمات المنقطعة', '$offlineCount', Icons.cancel, Colors.red),
            _buildStatCard('حالة التشفير والربط', 'آمن (HTTPS / API)', Icons.security, Colors.emerald),
          ],
        ),
      ),
    ];

    return Directionality(
      textDirection: TextDirection.rtl,
      child: Scaffold(
        appBar: AppBar(
          title: const Text('مراقب شبكة الميكروتيك'),
          backgroundColor: const Color(0xFF1E293B),
          actions: [
            Padding(
              padding: const EdgeInsets.symmetric(horizontal: 16),
              child: Center(
                child: Badge(
                  label: Text('$offlineCount'),
                  backgroundColor: Colors.red,
                  child: const Icon(Icons.notifications),
                ),
              ),
            )
          ],
        ),
        body: screens[_currentIndex],
        bottomNavigationBar: BottomNavigationBar(
          currentIndex: _currentIndex,
          onTap: (index) => setState(() => _currentIndex = index),
          backgroundColor: const Color(0xFF1E293B),
          selectedItemColor: Colors.emeraldAccent,
          unselectedItemColor: Colors.grey,
          items: const [
            BottomNavigationBarItem(icon: Icon(Icons.map), label: 'الخريطة الحية'),
            BottomNavigationBarItem(icon: Icon(Icons.warning), label: 'التنبيهات والمنقطعة'),
            BottomNavigationBarItem(icon: Icon(Icons.analytics), label: 'الإحصائيات'),
          ],
        ),
      ),
    );
  }

  Widget _buildStatCard(String title, String value, IconData icon, Color color) {
    return Card(
      color: const Color(0xFF1E293B),
      margin: const EdgeInsets.only(bottom: 12),
      child: ListTile(
        leading: Icon(icon, color: color, size: 30),
        title: Text(title, style: const TextStyle(color: Colors.grey, fontSize: 14)),
        trailing: Text(value, style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold, color: color)),
      ),
    );
  }
}
