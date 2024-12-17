# Flutter GetX Dummy Data Project

## Overview
This is a Flutter project demonstrating the use of GetX for state management and HTTP package for API integration. The application fetches and displays a list of dummy products from a JSON API.

## Prerequisites
- Flutter SDK
- Dart SDK

## Dependencies
Add the following dependencies to your `pubspec.yaml`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.6
  http: ^1.0.0
  get: ^4.6.6
```

## Project Structure
- `lib/`
  - `service/apiservice.dart`: API service for fetching data
  - `model/dummy_model.dart`: Data model for products
  - `controller/controller.dart`: GetX controller for state management
  - `main.dart`: Application entry point and main UI

## Code Examples

### Step 1. Create API Service (`apiservice.dart`)
```dart
class ApiService {
  // API Endpoint URL
  final String _url =
      'https://raw.githubusercontent.com/TheAppWizard/DummyData/refs/heads/main/dummy-api.json';

  /// Fetches data from the API
  Future<List<dynamic>> fetchData() async {
    try {
      final response = await http.get(Uri.parse(_url));

      if (response.statusCode == 200) {
        // Parse JSON response
        final List<dynamic> data = json.decode(response.body);
        return data;
      } else {
        throw Exception(
            'Failed to load data. Status code: ${response.statusCode}');
      }
    } catch (e) {
      throw Exception('Error fetching data: $e');
    }
  }
}
```
### Step 2. Create Response Model (`dummy_model.dart`)
[QuickType]([https://pub.dev/packages/get](https://app.quicktype.io)) for Creating Model
```dart
// To parse this JSON data, do
//
//     final dummyModel = dummyModelFromJson(jsonString);

import 'dart:convert';

DummyModel dummyModelFromJson(String str) => DummyModel.fromJson(json.decode(str));

String dummyModelToJson(DummyModel data) => json.encode(data.toJson());

class DummyModel {
  int statusCode;
  String statusMessages;
  Data data;

  DummyModel({
    required this.statusCode,
    required this.statusMessages,
    required this.data,
  });

  factory DummyModel.fromJson(Map<String, dynamic> json) => DummyModel(
    statusCode: json["statusCode"],
    statusMessages: json["statusMessages"],
    data: Data.fromJson(json["data"]),
  );

  Map<String, dynamic> toJson() => {
    "statusCode": statusCode,
    "statusMessages": statusMessages,
    "data": data.toJson(),
  };
}

class Data {
  List<Product> products;

  Data({
    required this.products,
  });

  factory Data.fromJson(Map<String, dynamic> json) => Data(
    products: List<Product>.from(json["products"].map((x) => Product.fromJson(x))),
  );

  Map<String, dynamic> toJson() => {
    "products": List<dynamic>.from(products.map((x) => x.toJson())),
  };
}

class Product {
  int id;
  String name;
  String description;
  double price;
  String currency;
  bool inStock;
  double rating;
  String imageUrl;

  Product({
    required this.id,
    required this.name,
    required this.description,
    required this.price,
    required this.currency,
    required this.inStock,
    required this.rating,
    required this.imageUrl,
  });

  factory Product.fromJson(Map<String, dynamic> json) => Product(
    id: json["id"],
    name: json["name"],
    description: json["description"],
    price: json["price"]?.toDouble(),
    currency: json["currency"],
    inStock: json["in_stock"],
    rating: json["rating"]?.toDouble(),
    imageUrl: json["image_url"],
  );

  Map<String, dynamic> toJson() => {
    "id": id,
    "name": name,
    "description": description,
    "price": price,
    "currency": currency,
    "in_stock": inStock,
    "rating": rating,
    "image_url": imageUrl,
  };
}

```

### Step 3. Integrate Model In Api Service (`apiservice.dart`)
```dart
import 'dart:convert';
import 'package:http/http.dart' as http;
import 'dummy_model.dart';

class ApiService {
  // API Endpoint URL
  final String _url =
      'https://raw.githubusercontent.com/TheAppWizard/DummyData/refs/heads/main/dummy-api.json';

  /// Fetches DummyModel data from the API
  Future<DummyModel> fetchData() async {
    try {
      final response = await http.get(Uri.parse(_url));

      if (response.statusCode == 200) {
        // Parse JSON response and convert to DummyModel
        final DummyModel dummyModel = dummyModelFromJson(response.body);
        return dummyModel;
      } else {
        throw Exception('Failed to load data. Status code: ${response.statusCode}');
      }
    } catch (e) {
      throw Exception('Error fetching data: $e');
    }
  }
}
```

### Step 4. GetX Controller (`controller.dart`)
```dart
import 'package:get/get.dart';
import '../model/dummy_model.dart';
import '../service/apiservice.dart';

class DummyController extends GetxController {
  final ApiService _apiService = ApiService();

  var isLoading = true.obs;

  ///Initial State of Model
  var dummyModel = DummyModel(
    statusCode: 0,
    statusMessages: '',
    data: Data(products: []),
  ).obs;

  var errorMessage = ''.obs;

  @override
  void onInit() {
    super.onInit();

    ///Call In Init if You want on page view
    fetchDummyData();
  }



  ///User This Method To Fetch Data
  void fetchDummyData() async {
    try {
      isLoading(true);
      final result = await _apiService.fetchData();
      dummyModel.value = result;
      errorMessage.value = '';
    } catch (e) {
      errorMessage(e.toString());
    } finally {
      isLoading(false);
    }
  }
}
```

### Step 5. Main View (`main.dart`)
```dart
import 'package:flutter/material.dart';
import 'package:get/get.dart';

import 'controller/controller.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});


  @override
  Widget build(BuildContext context) {
    return GetMaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Flutter Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home:  DummyView(),
    );
  }
}

class DummyView extends StatelessWidget {
  final DummyController controller = Get.put(DummyController());

   DummyView({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Dummy Product List',style: TextStyle(fontSize: 14),),
      ),
      body: Obx(() {


        if (controller.isLoading.value) {
          return const Center(
            child: CircularProgressIndicator(),
          );
        }


        ///Add Error Widgwt
        if (controller.errorMessage.isNotEmpty) {
          return Center(
            child: Text(
              'Error: ${controller.errorMessage}',
              style: const TextStyle(color: Colors.red),
            ),
          );
        }

        final products = controller.dummyModel.value.data.products;


        ///if Product Empty
        if (products.isEmpty) {
          return const Center(
            child: Text('No products found'),
          );
        }



        ///View
        return ListView.builder(
          itemCount: products.length,
          itemBuilder: (context, index) {
            final product = products[index];
            return Card(
              margin: const EdgeInsets.all(10),
              child: ListTile(
                leading: Image.network(
                  product.imageUrl,
                  width: 80,
                  height: 150,
                  fit: BoxFit.cover,
                  errorBuilder: (context, error, stackTrace) {
                    return const Icon(
                      Icons.broken_image,
                      size: 60,
                      color: Colors.grey,
                    );
                  },
                ),
                title: Text(product.name,style: const TextStyle(fontSize: 12),),
                subtitle: Text(
                  '${product.currency} ${product.price.toStringAsFixed(2)}',
                ),
                trailing: Icon(
                  product.inStock ? Icons.check_circle : Icons.cancel,
                  color: product.inStock ? Colors.green : Colors.red,
                ),
              ),
            );
          },
        );
      }),
    );
  }
}
```

## Results

![results](https://github.com/user-attachments/assets/38819ef0-7137-4c75-bd5a-6a002ae85f4b)

## Acknowledgments
- [GetX](https://pub.dev/packages/get) for state management
- [http](https://pub.dev/packages/http) for API requests

