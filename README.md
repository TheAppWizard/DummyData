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

### 1. API Service (`apiservice.dart`)
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

### 2. GetX Controller (`controller.dart`)
```dart
import 'package:get/get.dart';

import '../model/dummy_model.dart';
import '../service/apiservice.dart';

class DummyController extends GetxController {
  final ApiService _apiService = ApiService();

  var isLoading = true.obs;
  var dummyModel = DummyModel(
    statusCode: 0,
    statusMessages: '',
    data: Data(products: []),
  ).obs;

  var errorMessage = ''.obs;

  @override
  void onInit() {
    super.onInit();
    fetchDummyData();
  }

  /// Method to fetch data
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

### 3. Main Application (`main.dart`)
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
      home: DummyView(),
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
        title: const Text('Dummy Product List', style: TextStyle(fontSize: 14)),
      ),
      body: Obx(() {
        if (controller.isLoading.value) {
          return const Center(
            child: CircularProgressIndicator(),
          );
        }

        if (controller.errorMessage.isNotEmpty) {
          return Center(
            child: Text(
              'Error: ${controller.errorMessage}',
              style: const TextStyle(color: Colors.red),
            ),
          );
        }

        final products = controller.dummyModel.value.data.products;

        if (products.isEmpty) {
          return const Center(
            child: Text('No products found'),
          );
        }

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
                title: Text(product.name, style: const TextStyle(fontSize: 12)),
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

## Key Components

### API Service
- Fetches data from a GitHub raw JSON file
- Handles response parsing and error management

### GetX Controller
- Manages application state
- Initiates data fetching
- Handles loading and error states
- Provides reactive data for the UI

### User Interface
- Displays loading indicator
- Shows error messages
- Lists products with details
- Handles product images
- Indicates stock availability

## Features
- Fetch dummy product data from an API
- Reactive state management with GetX
- Error handling
- Image loading with fallback
- Responsive list view


## Acknowledgments
- [GetX](https://pub.dev/packages/get) for state management
- [http](https://pub.dev/packages/http) for API requests
