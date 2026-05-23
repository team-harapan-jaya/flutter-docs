# **Introduction**
**Database Configuration System using GetStorage**\
Pada section ini adalah cara penggunaan local storage secara expert dengan menggunakan simulasi database dan class, seperti backend pada Laravel, .NET, dan JS. Bertujuan membuat database secara center pada sebuah aplikasi mobile. supaya lebih mudah dalam pembacaan dan reusable.
# **Get Started**
- ## Storage Adapter
    Base interface read and write GetStorage
    ``` dart
    abstract class StorageAdapter {
        List<Map<String, dynamic>> read(String key);
        Future<void> write(String key, List<Map<String, dynamic>> data);
    }
    ```
- ## Get Storage Adapter
    Implementasi dari Storage Adapter
    ``` dart
    class GetStorageAdapter implements StorageAdapter {
        final box = GetStorage();

        @override
        List<Map<String, dynamic>> read(String key) {
            final data = box.read(key);
            if (data == null) return [];
            return List<Map<String, dynamic>>.from(data);
        }

        @override
        Future<void> write(String key, List<Map<String, dynamic>> data) async {
            await box.write(key, data);
        }
    }
    ```
- ## Entity Base
    Berisi class base untuk semua model pada table, yang nantinya semua class adalah hasil turunan dari class berikut
    ``` dart
    abstract class EntityBase {
        String get id;
    }
    ```
- ## Entity Mapper
    Berisi interface helper untuk mapping class
    ``` dart
    abstract class EntityMapper<T extends EntityBase> {
        Map<String, dynamic> toMap(T entity);
        T fromMap(Map<String, dynamic> map);
    }
    ```
- ## Get Storage Adapter
    Berisi inisialisasi database dan juga function untuk query database
    ``` dart
    abstract class StorageAdapter {
        List<Map<String, dynamic>> read(String key);
        Future<void> write(String key, List<Map<String, dynamic>> data);
    }
    ```
- ## Db Set
    Berisi fungsi utama inisialisasi database dan juga query
    ``` dart
    class DbSet<T extends EntityBase> {
        final String key;
        final EntityMapper<T> mapper;
        final StorageAdapter storage;

        late List<T> _data;

        DbSet({
            required this.key,
            required this.mapper,
            required this.storage
        }) {
            _load();
        }

        void _load() {
            final maps = storage.read(key);
            _data = maps.map(mapper.fromMap).toList();
        }

        List<T> toList() => List.unmodifiable(_data);

        Iterable<T> where(bool Function(T) predicate) => _data.where(predicate);

        T? firstOrNull(bool Function(T) predicate) {
            try {
                return _data.firstWhere(predicate);
            } catch (_) {
                return null;
            }
        }

        Future<void> add(T entity) async {
            _data.add(entity);
            await _save();
        }

        bool any([bool Function(T)? predicate]) {
            if (predicate == null) {
                return _data.isNotEmpty;
            }
            return _data.any(predicate);
        }

        Future<void> addRange(List<T> entities) async {
            _data.addAll(entities);
            await _save();
        }

        void update(T entity) {
            final index = _data.indexWhere((e) => e.id == entity.id);
            if (index != -1) {
                _data[index] = entity;
                _save();
            }
        }

        void remove(T entity) {
            _data.removeWhere((e) => e.id == entity.id);
            _save();
        }

        Future<void> _save() async {
            final maps = _data.map(mapper.toMap).toList();
            await storage.write(key, maps);
        }
    }
    ```
# **Implementation**
- ## **Storage Key**
    ``` dart
    class Storage {
        static const String transaction = "transaction";
        static const String prices = "prices";
        static const String transactionStationLogs = "transaction-station-logs";

        Future<void> clearAllStorage() async {
            final storage = GetStorage();
            
            await storage.remove(Storage.transaction);
            await storage.remove(Storage.transactionStationLogs);
            await storage.remove(Storage.originTransactionStationLogs);
        }

        bool hasValue(String key) {
                final storage = GetStorage();
                return storage.hasData(key);
            }
        }
    ```
- ## **Storage Key**
    ``` dart
    class Storage {
        static const String transaction = "transaction";
        static const String prices = "prices";
        static const String transactionStationLogs = "transaction-station-logs";

        Future<void> clearAllStorage() async {
            final storage = GetStorage();
            
            await storage.remove(Storage.transaction);
            await storage.remove(Storage.transactionStationLogs);
            await storage.remove(Storage.originTransactionStationLogs);
        }

        bool hasValue(String key) {
            final storage = GetStorage();
            return storage.hasData(key);
        }
    }
    ```
- ## **Entity**
    ``` dart
    class Price extends EntityBase {
        @override
        final String id;
        final String passengerId;
        final String passengerName;
        final String passengerCode;
        final String originStationId;
        final String originStationName;
        final String originStationCode;
        final String destinationStationId;
        final String destinationStationName;
        final String destinationStationCode;
        final double bruto;
        final double netto;
        final double commision;

        Price({
            required this.id,
            required this.passengerId,
            required this.passengerName,
            required this.passengerCode,
            required this.originStationId,
            required this.originStationName,
            required this.originStationCode,
            required this.destinationStationId,
            required this.destinationStationName,
            required this.destinationStationCode,
            required this.bruto,
            required this.netto,
            required this.commision,
        });
    }
    ```
- ## **Entity Mapper**
    ``` dart
    class PriceMapper extends EntityMapper<Price> {
        @override
        Price fromMap(Map<String, dynamic> map) {
            return Price(
            id: map['id'],
            passengerId: map['passengerId'],
            passengerName: map['passengerName'],
            passengerCode: map['passengerCode'],
            originStationId: map['originStationId'],
            originStationName: map['originStationName'],
            originStationCode: map['originStationCode'],
            destinationStationId: map['destinationStationId'],
            destinationStationName: map['destinationStationName'],
            destinationStationCode: map['destinationStationCode'],
            bruto: map['bruto'],
            netto: map['netto'],
            commision: map['commision']
            );
        }

        @override
        Map<String, dynamic> toMap(Price entity) {
            return {
            'id': entity.id,
            'passengerId': entity.passengerId,
            'passengerName': entity.passengerName,
            'passengerCode': entity.passengerCode,
            'originStationId': entity.originStationId,
            'originStationName': entity.originStationName,
            'originStationCode': entity.originStationCode,
            'destinationStationId': entity.destinationStationId,
            'destinationStationName': entity.destinationStationName,
            'destinationStationCode': entity.destinationStationCode,
            'bruto': entity.bruto,
            'netto': entity.netto,
            'commision': entity.commision
            };
        }
    }
    ```