# **Harapan Jaya Flutter Docs**

Dokumen ini berisi standar dan pedoman pengembangan aplikasi Flutter yang digunakan dalam tim, mencakup arsitektur, struktur folder, penamaan kode, serta pengelolaan data. Tujuannya adalah untuk memastikan konsistensi, meningkatkan kualitas kode, serta mempermudah proses pengembangan dan pemeliharaan aplikasi.


# **Folder Structure**
Struktur folder pada proyek Flutter ini mengadopsi konsep Domain-Driven Design (DDD) untuk memisahkan tanggung jawab setiap layer secara jelas dan menjaga kode tetap terorganisir, scalable, serta mudah untuk dikembangkan. Struktur utama terdiri dari empat layer utama, yaitu:

**Application, Domain, Infrastructure, dan Presentation**

- ## Application Layer
  **Application** bertanggung jawab untuk mengatur alur bisnis aplikasi (business flow) tanpa berisi logic inti domain.\
    Beberapa tanggung jawab utama:
    - Meng-handle use case atau fitur aplikasi
    - Mengatur interaksi antara layer Presentation dan Domain
    - Mengelola state (misalnya menggunakan Bloc, Cubit, atau state management lainnya)
    - Berisi DTO (Data Transfer Object) jika diperlukan

- ## Domain Layer
  **Domain** merupakan inti dari aplikasi yang berisi business logic utama.\
    Beberapa isi dari layer ini:
    - Entity
    - Value Object
    - Enum
    - Business rules / logic utama

- ## Infrastructure Layer
  **Infrastructure** berisi implementasi dari hal-hal teknis yang dibutuhkan oleh aplikasi.
    - Implementasi Repository (yang sebelumnya didefinisikan di Domain)
    - Data source (API, database lokal seperti SQLite, dll)
    - Konfigurasi network (misalnya Dio)
    - Service eksternal

- ## Presentation Layer
  **Presentation** bertanggung jawab untuk tampilan (UI) dan interaksi dengan user.\
    Beberapa isi dari layer ini.
    - Halaman (pages / screens)
    - Widget
    - State management (controller)

# **Naming Conventions**
Standarisasi penamaan digunakan untuk menjaga konsistensi kode, meningkatkan readability, serta mempermudah kolaborasi dalam tim. Aturan ini mencakup penamaan class, enum, page,
widget, dan interface.
- ## Class dan Enum
    Gunakan PascalCase untuk penamaan class dan enum Nama harus merepresentasikan tujuan atau fungsi dengan jelas
    ``` dart
    class User {}
    enum PaymentStatus {}
    ```
- ## Pages
    Gunakan suffix Page untuk setiap halaman dan Gunakan PascalCase\
    Page harus berupa turunan dari StatelessWidget atau StatefulWidget
    ``` dart
    class HomePage extends StatelessWidget {}
    class CameraListenerPage extends StatefulWidget {}
    ```
- ## Widget berbasis Class
    Gunakan suffix Widget dan Gunakan PascalCase\
    Digunakan untuk reusable component atau widget kompleks
    ``` dart
    class ProfileWidget extends StatelessWidget {}
    ```
- ## Widget berbasis Function
    Gunakan camelCase dan Digunakan untuk widget sederhana atau bagian kecil dari UI
    ``` dart
    Widget profileWidget() {
        return const SizedBox();
    }
    ```
- ## Interface Abstract Class
    Gunakan PascalCase dan Tidak perlu menggunakan prefix
    `I`\
    Gunakan kata yang merepresentasikan kontrak/service
    ``` dart
    abstract class ProfileService {}
    ```
- ## Implementasi Interface
    Gunakan suffix
    `Impl`
    dan untuk class implementasi Gunakan PascalCase
    ``` dart
    class ProfileServiceImpl extends ProfileService {}
    ```

# **Json Serialization (Dto)**
Untuk proses serialisasi dan deserialisasi JSON, proyek ini menggunakan library
**Freezed**
sebagai standar utama.\
DTO (Data Transfer Object) digunakan untuk merepresentasikan response dari API,\
dengan aturan utama: `Tipe data pada DTO harus sesuai dengan tipe data dari API.`
Tidak diperbolehkan mengubah tipe data (misalnya: API mengirim DateTime, tetapi di Flutter
menggunakan String).
- ## Integer
    - Tidak menggunakan nullable
    - Wajib memiliki default value 0
    - Digunakan untuk angka kecil seperti: jumlah pesanan, quantity
    ``` dart
    @JsonKey(name: 'quantity') @Default(0) int quantity
    ```
- ## Double
    - Tidak menggunakan nullable
    - Wajib memiliki default value 0
    - Digunakan untuk angka besar seperti harga, total pembayaran
    ``` dart
    @JsonKey(name: 'price') @Default(0) double price
    ``` 
- ## Boolean
    - Tidak menggunakan nullable
    - Wajib memiliki default value false
    ``` dart
    @JsonKey(name: 'isActive') @Default(false) bool isActive
    ``` 
- ## DateTime
    - Gunakan tipe DateTime untuk data tanggal dari API
    - Tidak diperbolehkan menggunakan String untuk representasi tanggal
    ``` dart
    @JsonKey(name: 'createdAt') DateTime? createdAt
    ``` 
- ## Enum
    - Gunakan enum untuk data status yang berasal dari API (biasanya dalam bentuk String)
    - Wajib membuat converter sendiri untuk mapping dari String ke Enum\
    Contoh class `enum`
    ``` dart
    enum PaymentStatus {
        unpaid,
        settlement,
        expire,
        failed
    }
    ``` 
    Contoh `enum converter`
    ``` dart
    PaymentStatus _paymentStatusFromJson(dynamic value) {
        try {
            return PaymentStatus.values.firstWhere(
                (e) => e.name.toLowerCase() == value.toString().toLowerCase(),
                orElse: () => PaymentStatus.unpaid,
            );
        } catch (e) {
            return PaymentStatus.unpaid;
        }
    }
    ``` 
    Contoh `penerapan pada freezed`
     ``` dart
    @JsonKey(name: 'status', fromJson: _paymentStatusFromJson) @Default(PaymentStatus.unpaid) PaymentStatus paymentStatus,
    ```
- ## Object
    - Gunakan class dari freezed
    - Buat nullable jika memang null
    - Buat dengan default value jika tidak null\
    Contoh jika `nullable`
    ``` dart
    @JsonKey(name: 'type') VoucherDetailPropertyTypeReadDto? type
    ``` 
    Contoh jika `tidak nullable`
    ``` dart
    @JsonKey(name: 'type') @Default(VoucherDetailPropertyTypeReadDto()) VoucherDetailPropertyTypeReadDto type
    ``` 
- ## List
    - Gunakan class dari freezed
    - Wajib dengan default value `[]`
    ``` dart
    @JsonKey(name: 'properties') @Default([]) List<VoucherDetailPropertyReadDto> properties,
    ``` 

# **Entity (Opsional tapi sangat disarankan)**
DTO tidak boleh langsung digunakan di layer Presentation (disarankan dimapping ke Domain jika diperlukan),
Entity bisa didapatkan dari Dto yang di convert menjadi entity\
- ## Contoh Sederhana Class Entity
    ``` dart
    class User {
        final String id;
        final String name;
        final String email;
        final String? address;
        final UserStatus status;

        User({
            required this.id,
            required this.name,
            required this.email,
            required this.status,
            this.address
        });
    }
    ```
- ## Contoh Convert dari DTO ke Entity :
    ``` dart
    @freezed
    class UserReadDto with  _$UserReadDto{
    const factory UserReadDto({
        @JsonKey(name: 'Id') @Default("00000000-0000-0000-0000-000000000000") String id,
        @JsonKey(name: 'Name') @Default("") String name,
        @JsonKey(name: 'Email') @Default("") String email,
        @JsonKey(name: 'Status') @Default("") String status,
        @JsonKey(name: 'Address') String? address,
    }) = _UserReadDto;

    factory UserReadDto.fromJson(Map<String, dynamic> json) => _$UserReadDtoFromJson(json);
    }

    extension UserReadDtoMapping on UserReadDto {
        User toEntity() {
            return User(
                id : id,
                name : name,
                email : email,
                status : _parseUserStatus(status),
                address : address
            );
        }
    }
    ```
- ## Contoh Penggunaan `toEntity()` :
    ``` dart
    final result = MySnackSummaryReadDto.fromJson(response.data);
    return Right(result.toEntity());
    ```
# **API Communication (CRUD)**
Untuk proses Create, Read, Update, dan Delete (CRUD), proyek ini menggunakan library `Dio` sebagai standar utama dalam melakukan komunikasi dengan API.
Pendekatan yang digunakan bertujuan untuk menjaga konsistensi, mempermudah handling error, serta memastikan kode tetap terstruktur dan mudah di-maintain.\
Selalu gunakan versio dio terbaru pada url [Dio](https://pub.dev/packages/dio)
**Standar yang Digunakan**
- ## HTTP Client (Dio)
    - Seluruh komunikasi API wajib menggunakan Dio
    - Konfigurasi seperti base URL, header, dan interceptor dikelola secara terpusat
    ``` dart
    class DioClient {
        static Dio create() {
            final dio = Dio(
            BaseOptions(
                    connectTimeout: Duration(seconds: 45),
                    receiveTimeout: Duration(seconds: 45)
                )
            );

            //add interceptor
            dio.interceptors.addAll([
                PrettyDioLogger(requestBody: true),
                DioTimingInterceptor()
            ]);

            return dio;
        }
    }
    ``` 
- ## HTTP Client Interceptor (Dio Interceptor)
    - Menggunakan `pretty_dio_logger` untuk memudahkan dalam melihat hasil request, `pretty_dio_logger 1.4.0` bisa diambil di url [pretty_dio_logger](https://pub.dev/packages/pretty_dio_logger)
    ``` dart
    dio.interceptors.add(PrettyDioLogger(requestBody: true));
    ```
    - Menggunakan package `synchronized` pada url [synchronized](https://pub.dev/packages/synchronized) untuk handler Refresh Token\
    dengan contoh Refresh Token Interceptor seperti berikut :
    ``` dart
    class DioRefreshTokenInterceptor extends QueuedInterceptor {
        final Dio dio;
        final Lock _lock = Lock();

        DioRefreshTokenInterceptor(this.dio);

        @override
        void onError(DioException err, ErrorInterceptorHandler handler) async {
            //unauthorized handler
            if (err.response?.statusCode == 401) {
            var accessToken = await TokenService().readAccessToken();
            // Token has expired, refresh it
            try {
                await _lock.synchronized(() async {
                if (err.requestOptions.headers['Authorization'] != 'Bearer $accessToken') {
                    // Token has already been refreshed by another request
                    handler.resolve(await dio.fetch(err.requestOptions..headers['Authorization'] = 'Bearer $accessToken'));
                    return;
                }
                
                // Perform token refresh
                var refreshToken =  await TokenService().refreshToken();

                if (refreshToken.isSuccessStatusCode) {
                    //get new access token
                    var newAccessToken = await TokenService().readAccessToken();
                    // Retry the original request with the new token
                    err.requestOptions.headers['Authorization'] = 'Bearer $newAccessToken';
                    handler.resolve(await dio.fetch(err.requestOptions));
                } else {
                    await TokenService().handleErrorRefreshToken();
                    return;
                }
                });
            } catch (e) {
                // If refresh fails, propagate the error
                handler.next(err);
            }
            return;
            }
            // For other errors, proceed normally
            handler.next(err);
        }
    }
    ```
    Penggunaan pada interceptor adalah sebagai berikut :
    ``` dart
    dio.interceptors.add(DioRefreshTokenInterceptor(dio));
    ```
- ## HTTP Client Exception Handler (Dio Interceptor Handler)
    - Menggunakan custom handler berdasarkan exception yang diberikan oleh Http Response dari Dio, Seperti Bad Request, NotFound dll\
    Contoh Exception Handler :
    ``` dart
    class ApiFailure {
        final int statusCode;
        final Response? response;
        final DioExceptionType? type;
        final String? customMessage;

        ApiErrorType get errorType => _getErrorType(statusCode, response, type);
        String get message => _getErrorMessage(_getErrorType(statusCode, response, type), response, customMessage);

        ApiFailure({
            required this.statusCode,
            this.response,
            this.type,
            this.customMessage
        });
        }

        String _getErrorMessage(ApiErrorType errorType, Response? response, String? customMessage) {
            switch (errorType) {
                case ApiErrorType.unauthorized:
                    return ApiErrorMessage.unauthorized;
                case ApiErrorType.forbidden:
                    return ApiErrorMessage.forbiden;
                case ApiErrorType.timeout:
                    return ApiErrorMessage.timeout;
                case ApiErrorType.noInternetConnection:
                    return ApiErrorMessage.noInternetConnection;
                case ApiErrorType.methodNotAllowed:
                    return ApiErrorMessage.methodNotAllowed;
                case ApiErrorType.notFound:
                    return _parseNotFoundMessage(response);
                case ApiErrorType.serverError:
                    return ApiErrorMessage.internalServerError;
                case ApiErrorType.badRequest:
                    return _parseBadRequestMessage(response, customMessage);
                default:
                    return ApiErrorMessage.undefined;
            }
        }

        String _parseNotFoundMessage(Response? response) {
        if (response == null || response.data == null) {
            return ApiErrorMessage.notFound;
        }

        final data = response.data;
        if (data is! Map<String, dynamic>) {
            return ApiErrorMessage.notFound;
        }

        return data['Detail'] ?? ApiErrorMessage.notFound;
        }

        String _parseBadRequestMessage(Response? response, String? customMessage) {
        if (!customMessage.isNullOrWhitespace) {
            return customMessage!;
        }

        if (response == null || response.data == null) {
            return ApiErrorMessage.undefined;
        }

        final data = response.data;
        if (data is! Map<String, dynamic>) {
            return ApiErrorMessage.undefined;
        }

        String message = data['Detail'] ?? customMessage ?? ApiErrorMessage.undefined;
        if (_isValidationError(data)) {
            return _getValidationErrorMessage(data);
        }

        return message;
        }

        bool _isValidationError(dynamic data) {
        if ((data['errors'] is Map || data['Errors'] is Map)) {
            return true;
        }
        if (data['detail'] != null || data['Detail'] != null) {
            String detail = data['detail'] ?? data['Detail'];
            return detail.toLowerCase().contains("errors") || detail.toLowerCase().contains("validation");
        }
        return false;
        }

        String _getValidationErrorMessage(dynamic data) {
        //handle error validation with 'Data' response
        var errorData = data['errors'] ?? data['Data'] ?? data['data'];
        if (errorData is Map<String, dynamic>) {
            return errorData.entries.map((entry) {
            String field = entry.key;
            String messages = (entry.value is List) ? entry.value.join(', ') : entry.value.toString();
            return "$field: $messages";
            }).join('\n');
        }

        if (errorData is List<dynamic>) {
            return errorData.map((element) => element['Message']?.toString() ?? '')
            .where((msg) => msg.isNotEmpty)
            .join(', ');
        }
        
        if (data['errors'] is Map<String, dynamic> || data['Errors'] is Map<String, dynamic>) {
            var errors = (data['errors'] ?? data['Errors']) as Map<String, dynamic>;
            List<String> errorMessages = [];

            errors.forEach((field, messages) {
            errorMessages.add("$field: ${messages.join(', ')}");
            });

            return errorMessages.join("\n");
        }
        return ApiErrorMessage.validation;
        }

        ApiErrorType _getErrorType(int statusCode, Response? response, DioExceptionType? type) {
        //catch timeout exception
        if (type is DioExceptionType) {
            if (type == DioExceptionType.receiveTimeout || type == DioExceptionType.sendTimeout || type == DioExceptionType.connectionTimeout) return ApiErrorType.timeout;
            if (type == DioExceptionType.connectionError) return ApiErrorType.noInternetConnection;
        }

        if (statusCode == 0) {
            return ApiErrorType.unknown; // Biasanya 0 digunakan untuk koneksi timeout/offline
        } else if (statusCode >= 400 && statusCode < 500) {
            if (statusCode == 401) return ApiErrorType.unauthorized;
            if (statusCode == 403) return ApiErrorType.forbidden;
            if (statusCode == 404) return ApiErrorType.notFound;
            if (statusCode == 405) return ApiErrorType.methodNotAllowed;
            return ApiErrorType.badRequest;
        } else if (statusCode >= 500) {
            return ApiErrorType.serverError;
        } else {
            return ApiErrorType.unknown;
        }
    }
    ```
    Contoh Penggunaan `ApiFailure`
    ``` dart
    on DioException catch (e) {
        return Left(
            ApiFailure(type: e.type, statusCode: e.response?.statusCode ?? 0, response: e.response)
        );
    }
    ```
- ## Pattern Either (Success / Failure)
    - Gunakan konsep Either untuk membedakan hasil sukses dan gagal
    - `Left` digunakan untuk error / failure
    - `Right` digunakan untuk success / data
    - Menggunakan package `fpdart` disini [fpdart](https://pub.dev/packages/fpdart)\
    Contoh Implementasi
    ``` dart
    @override
    Future<Either<ApiFailure, Trayek>> getTrayekAsync(String id) async {
        String url = "${baseUrl}Trayeks/$id/Preview";
        try {
            final response = await dio.get(
                url,
                options: Options(
                headers: {
                        "Authorization" : "Token ${CurrentUser.token}"
                    }
                )
            );

            var result = TrayekReadDto.fromJson(response.data).toEntity();
            return Right(result);
        } on DioException catch (e) {
            return Left(
                ApiFailure(type: e.type, statusCode: e.response?.statusCode ?? 0, response: e.response)
            );
        } catch (e) {
            return Left(
                ApiFailure(statusCode: 400)
            );
        }
    }
    ```
    Contoh penggunaan Logika `Left` dan `Right` di controller
    ``` dart
    var trayek = await _trayekRepository.getTrayekAsync(id);
    trayek.match((failure) {
        //Handle failed
    }, (trayek) {
        //Handle success
    });
    ```
- ## Interface dan Implementasi
    - Gunakan interface (abstract class) sebagai kontrak
    - Gunakan class implementasi untuk logic sebenarnya
    - Memudahkan testing dan dependency injection

    **Contoh Implementasi**
    **Interface**
    ``` dart
    abstract class TrayekRepository {
        Future<Either<ApiFailure, Trayek>> getTrayekAsync(String id);
    }
    ```
    **Implementasi**
    ``` dart
    class TrayekRepositoryImpl extends TrayekRepository {
        final Dio dio;
        final String baseUrl;

        TrayekRepositoryImpl(this.dio) : baseUrl = "${BaseUrl.parent}${ApiUrl.managementV2}";

        @override
        Future<Either<ApiFailure, Trayek>> getTrayekAsync(String id) async {
            String url = "${baseUrl}Trayeks/$id/Preview";
                try {
                    final response = await dio.get(
                        url,
                        options: Options(
                        headers: {
                            "Authorization" : "Token ${CurrentUser.token}"
                        }
                        )
                    );

                    var result = TrayekReadDto.fromJson(response.data).toEntity();
                    return Right(result);
                } on DioException catch (e) {
                    return Left(ApiFailure(type: e.type, statusCode: e.response?.statusCode ?? 0, response: e.response));
                } catch (e) {
                return Left(ApiFailure(statusCode: 400));
                }
            }
        }
    ```
- ## HTTP GET
    - Query parameter tidak boleh ditulis langsung di URL
    - Gunakan parameter `queryParameters` yang disediakan oleh Dio
    - Bertujuan agar URL tetap bersih dan mudah dibaca
    ``` dart
    final response = await dio.get(
        '/orders',
        queryParameters: {
            'pageNumber': 1,
            'pageSize': 10,
        },
    );
    ```
- ## HTTP POST
    - Body tidak boleh menggunakan `dynamic`
    - Wajib menggunakan class (DTO), baik dari Freezed maupun class biasa
    - Wajib menggunakan Content-Type: application/json
    ``` dart
    final request = CreateOrderDto(
        name: 'Order 1',
        quantity: 2,
    );
    final response = await dio.post(
        '/orders',
        data: request.toJson(),
        options: Options(
            contentType: Headers.jsonContentType,
        ),
    );
    ```
- ## HTTP PUT
    - Body tidak boleh menggunakan `dynamic`
    - Wajib menggunakan class (DTO), baik dari Freezed maupun class biasa
    - Wajib menggunakan Content-Type: application/json
    ``` dart
    final request = CreateOrderDto(
        name: 'Order 1',
        quantity: 2,
    );
    final response = await dio.put(
        '/orders/1',
        data: request.toJson(),
        options: Options(
            contentType: Headers.jsonContentType,
        ),
    );
    ```
- ## HTTP PATCH
    - Body tidak boleh menggunakan `dynamic`
    - Wajib menggunakan class (DTO), baik dari Freezed maupun class biasa
    - Wajib menggunakan Content-Type: application/json
    ``` dart
    final request = UpdateOrderStatusDto(
        status: 'paid',
    );
    final response = await dio.put(
        '/orders/1/Status',
        data: request.toJson(),
        options: Options(
            contentType: Headers.jsonContentType,
        ),
    );
    ```
- ## ⚠️ WARNING ⚠️
Setiap request dan response wajib memiliki model yang jelas (DTO).\
Tidak diperbolehkan menggunakan ❌`Map<String, dynamic>`❌ secara langsung dalam proses komunikasi data, terutama di luar layer Infrastructure.\
Penggunaan DTO bertujuan untuk:
    - Menjaga struktur data tetap konsisten
    - Menghindari kesalahan parsing data
    - Meningkatkan readability dan maintainability kode
    - Mempermudah proses debugging dan refactoring\
`Contoh yang salah ❌`
``` dart
Map<String, dynamic> body = {
        "productId" : 1,
        "quantity"  : 10,
        "paymentType" : "cash"
    };
```
`Contoh yang benar ✅`
``` dart
class Checkout {}
var checkout = Checkout(
        id : 1,
        quantity : 10,
        paymentType : "cash"
    );
```

# **Environment Configuration (Flutter Flavor)**
Untuk pengelolaan environment (seperti **development, staging, dan production**), proyek ini menggunakan package [flutter_flavor](http://pub.dev/packages/flutter_flavor) sebagai standar utama.
Penggunaan Flutter Flavor bertujuan untuk memisahkan konfigurasi aplikasi berdasarkan environment, sehingga memudahkan pengelolaan URL API, konfigurasi aplikasi, serta proses build dan deployment.\
## Tujuan Penggunaan Flavor
    - Memisahkan konfigurasi berdasarkan environment
    - Menghindari perubahan manual pada kode saat berpindah environment
    - Meminimalisir risiko kesalahan (misalnya penggunaan API production di development)
    - Mendukung proses build yang lebih terstruktur
## Standar Environment
    - Development (dev) → digunakan untuk proses development
    - Production (prod) → digunakan untuk aplikasi yang dirilis ke user
## Contoh Konfigurasi
- Konfigurasi base untuk `flavor_config`
    ``` dart
    class FlavorConfig {
        final String flavor;
        final String urlSuperApps;
        final ThemeMode theme;

        static late FlavorConfig _instance;

        factory FlavorConfig({
        required String flavor,
        required String urlSuperApps,
        required ThemeMode theme
        }) {
        _instance = FlavorConfig._internal(flavor, urlSuperApps, theme);
        return _instance;
        }

        FlavorConfig._internal(this.flavor, this.urlSuperApps, this.theme);

        static FlavorConfig get instance => _instance;
    }
    ```
## Contoh penggunaan pada `main_dev`
    ``` dart
    void main() async {
        FlavorConfig(
            flavor: "development",
            urlSuperApps: "https://dev-api.harapan-jaya.com/",
            theme: ThemeType.Dark
        );
        runApp(const MyApp());
    }
    ```
## Contoh penggunaan pada `main_prod`
    ``` dart
    void main() async {
        FlavorConfig(
            flavor: "production",
            urlSuperApps: "https://api.harapan-jaya.com/",
            theme: ThemeType.Dark
        );
        runApp(const MyApp());
    }
    ```
## Contoh pemanggilan variable
    ``` dart
    static String parent = FlavorConfig.instance.urlSuperApps;
    ```
## Pengaturan tambahan
    pada build gradle supaya memudahkan dalam build apk ataupun appbundle
    ``` java
    flavorDimensions "default"
    productFlavors {
        development {
            dimension "default"
            applicationIdSuffix ".dev"
            versionNameSuffix "-dev"
            resValue "string", "app_name", "(dev) Harapan Jaya Prima"
        }

        production {
            dimension "default"
            resValue "string", "app_name", "Harapan Jaya Prima"
        }
    }
    ```
## Pengaturan tambahan
    jika menggunakan vscode, pada bagian `launch.json`
    ``` json
    {
        "version": "0.2.0",
        "configurations": [
            {
                "name": "Flutter Dev",
                "request": "launch",
                "type": "dart",
                "program": "lib/main_dev.dart",
                "toolArgs": [
                "--flavor",
                "development"
                ],
                "args": [
                    "--flavor",
                    "development"
                ]
            },
            {
                "name": "Flutter Prod",
                "request": "launch",
                "type": "dart",
                "program": "lib/main_prod.dart",
                "toolArgs": [
                "--flavor",
                "production"
                ],
                "args": [
                    "--flavor",
                    "production"
                ]
            }
        ]
    }
    ```

# **Controller**
Pada layer Controller, developer diperbolehkan menggunakan state management apapun seperti **GetX, Bloc**, atau **lainnya**. Namun, untuk menjaga konsistensi dan kualitas kode, terdapat standar dalam penulisan controller yang harus diikuti. Pada dokumentasi ini, contoh implementasi menggunakan **GetX**.

**Prinsip Umum**\
Controller berfungsi sebagai penghubung antara Presentation dan Application/Domain, serta bertanggung jawab dalam mengelola state dan alur data.\
`Controller hanya berisi logic, dan tidak boleh menangani hal-hal yang berkaitan langsung dengan UI.`

**Standar Penulisan Controller**
## Menggunakan Interface Repository
- Controller harus bergantung pada interface (abstraction), bukan implementasi langsung
- Bertujuan untuk menjaga loose coupling dan memudahkan testing

Contoh yang kurang tepat ❌
``` dart
var checkout = await CheckoutRepository().checkoutAsync(body) // ❌ Tidak diperbolehkan
```
Contoh yang tepat ✅
``` dart
class TrayekController extends GetxController {
    final TrayekRepository _trayekRepository;

    TrayekController(this._trayekRepository);

    void _getTrayekAsync(String id) async {
        var trayek = await _trayekRepository.getTrayekAsync(id);
        trayek.match((failure) {
            //handle error
        }, (trayek) {
            //handle success
        });
    }
}
```
## Tidak Mengandung Logic UI
- Tidak boleh Melakukan navigasi (contoh: `Get.back()`, `Get.to()`)
- Tidak Menampilkan dialog (contoh: dialog error, loading, dll)
- Tidak boleh menampilkan snackbar
**Semua hal terkait UI harus ditangani di layer Presentation.**\
Contoh yang kurang tepat ❌
``` dart
Future<void> submitOrder() async {
    final result = await _repository.createOrder();

    result.fold(
        (error) {
            Get.snackbar('Error', error); // ❌ Tidak diperbolehkan
        },
        (data) {
            Get.back(); // ❌ Tidak diperbolehkan
        },
    );
}
```
Contoh yang tepat ✅\
**Controller:**
``` dart
Future<Either<String, OrderDto>> submitOrder() async {
    return await _repository.createOrder();
}
```
**Presentation (UI):**
``` dart
final result = await controller.submitOrder();
result.fold(
    (error) {
        Get.snackbar('Error', error); // ✅ di UI layer
    },
    (data) {
        Get.back(); // ✅ di UI layer
    },
);
```

## Pemisahan Function Public dan Private
- Gunakan method public untuk fungsi yang dipanggil dari UI
- Gunakan method private `(_)` untuk logic internal
**Fungsi Public**
``` dart
//Fungsi
Future<bool> isValidCartData(Cart cart) {}

//Penggunaan
_cartController.isValidCartData(Cart cart);
```
**Fungsi Private**
``` dart
void _initCheckoutController() {}
```

## Return Value yang Jelas
- Fungsi dalam controller sebaiknya memiliki return value yang jelas
- Hindari fungsi tanpa return (void) jika mengandung proses penting
``` dart
bool isValidCart(List<Cart> carts) {
    return carts.length() > 0;
}
```

## Pembagian Controller
Controller dapat dibagi menjadi beberapa bagian berdasarkan fitur atau use case untuk menghindari penumpukan logic dalam satu controller.\
Tidak semua fungsi harus ditempatkan dalam satu controller utama.\
**Contoh :**
- `CartController` → mengelola data cart (add, update, remove item)
- `CartCheckoutController` → mengelola proses checkout
- `CartHistoryController` → mengelola riwayat transaksi
Tujuan dari pembagian ini:
- Menghindari controller yang terlalu besar (God Controller)
- Mempermudah maintenance dan pengembangan fitur
- Meningkatkan keterbacaan kode
**Catatan Penting**
- Setiap controller harus memiliki tanggung jawab yang jelas
- Hindari overlap logic antar controller
- Hindari pembagian controller yang terlalu kecil tanpa alasan yang kuat
- Gunakan penamaan berbasis fitur atau use case, bukan berdasarkan teknis
    ### ❌ Semua Logic di Satu Controller (God Controller) ❌
    ``` dart
    class CartController extends GetxController {
        // CART
        Future<void> getCart() async {
            //
        }
        // CHECKOUT
        Future<void> checkout() async {
            //
        }
        // HISTORY
        Future<void> getHistory() async {
            //
        }
    }
    ```
    ### ✅ Dipisah ke Beberapa Controller ✅
    ``` dart
    class CartController extends GetxController {
        // CART
        Future<void> getCart() async {
            //
        }
    }
    class CartCheckoutController extends GetxController {
        // CHECKOUT
        Future<void> checkout() async {
            //
        }
    }
    class CartHistoryController extends GetxController {
        // HISTORY
        Future<void> getHistory() async {
            //
        }
    }
    ```
    ### ⚠️ Catatan Tambahan ⚠️
    Pemisahan controller menjadi beberapa bagian tidak berarti semua controller harus diinisialisasi atau dipanggil di awal.\
    Controller harus hanya digunakan saat dibutuhkan (on-demand) sesuai dengan konteks fitur atau halaman yang sedang aktif.
