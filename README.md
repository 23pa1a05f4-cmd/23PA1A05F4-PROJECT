<img width="688" height="1536" alt="home_screen" src="https://github.com/user-attachments/assets/0d9115d2-5aee-4c3c-923f-6eb1baed12a3" />
# 🌍 Country Explorer – Flutter Widget Testing

A modern and interactive Country Explorer application developed using Flutter and Dart. The application fetches live data from the REST Countries API and displays each country's flag, population, languages, region and capital, with live search and region filtering.

This project also demonstrates Flutter Widget Testing (using mock data) to verify list display, search, region filtering, empty state and error state handling.

## 📌 Project Overview

- **Project Name:** Country Explorer
- **Tagline:** Explore the World, One Country at a Time
- **Framework:** Flutter
- **Programming Language:** Dart
- **Testing Framework:** Flutter Widget Testing
- **Platform:** Android / Web
- **API Used:** REST Countries API (`https://restcountries.com/v3.1`)
- **SDG:** SDG 4 – Quality Education

## ✨ Features

- Modern Material 3 user interface.
- Live data fetched from the REST Countries API.
- Country cards showing flag, name, region, population and languages.
- Case-insensitive search by country name (updates on every keystroke).
- Region filter chips (All, Africa, Americas, Asia, Europe, Oceania, Antarctic).
- Search and region filter work together.
- Country detail view with large flag, capital, region, population and languages.
- Loading indicator while data is being fetched.
- Error message with Retry button.
- "No countries found" empty state.
- Flag images with loading and error fallbacks.
- Responsive user interface.
- Automated widget testing with a mocked data source.

## 🌐 REST Countries API

**Endpoint used:**

```
GET https://restcountries.com/v3.1/all?fields=name,flags,population,languages,region,capital,cca3
```

The `fields` query parameter is required for the `/all` endpoint and keeps the response small and fast.

| JSON Field | Meaning | Used For |
|---|---|---|
| `name.common` | Common country name | Card title and search |
| `flags.png` | Flag image URL | Flag on card and detail page |
| `population` | Integer population | Population display |
| `languages` | Map of code to language name | Languages display |
| `region` | Continent-level region | Region filter |
| `capital` | List of capital cities | Detail page |
| `cca3` | Three-letter country code | Unique ID |

## 🔄 Application Flow

```
App start → fetchCountries() → HTTP GET → JSON decoded to List<Country>
→ setState() → list shown

User types in search / taps a region chip → query / selectedRegion updated
→ filtered list recomputed → UI rebuilds with matching countries
```

## 🖥️ Application Screenshots

### 1. Home Screen with Country List
![Home Screen](<img width="250" alt="home_screen" src="https://github.com/user-attachments/assets/0d9115d2-5aee-4c3c-923f-6eb1baed12a3" />)

### 2. Region Filter Applied
![Region Filter](screenshots/region_filter.png)

### 3. Search Results (Indonesia)
![Search Results](screenshots/search_result.png)

### 4. Country Detail Screen
![Country Detail](screenshots/country_detail.png)

## 💻 Source Code

### 1. Country Model
**File:** `lib/models/country.dart`

```dart
class Country {
  final String code;
  final String name;
  final String flag;
  final String region;
  final String capital;
  final int population;
  final List<String> languages;

  Country({
    required this.code,
    required this.name,
    required this.flag,
    required this.region,
    required this.capital,
    required this.population,
    required this.languages,
  });

  factory Country.fromJson(Map<String, dynamic> json) {
    final langs = json['languages'] as Map<String, dynamic>?;
    final caps = json['capital'] as List<dynamic>?;

    return Country(
      code: json['cca3'] ?? '',
      name: json['name']?['common'] ?? 'Unknown',
      flag: json['flags']?['png'] ?? '',
      region: json['region'] ?? 'Unknown',
      capital: (caps != null && caps.isNotEmpty) ? caps.first : 'N/A',
      population: json['population'] ?? 0,
      languages: langs?.values.map((e) => e.toString()).toList() ?? [],
    );
  }
}
```

### 2. Fetching Data from the API
**File:** `lib/services/country_service.dart`

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;
import '../models/country.dart';

Future<List<Country>> fetchCountries() async {
  final uri = Uri.parse(
    'https://restcountries.com/v3.1/all'
    '?fields=name,flags,population,languages,region,capital,cca3',
  );

  final response = await http.get(uri);

  if (response.statusCode != 200) {
    throw Exception('Failed to load countries');
  }

  final List<dynamic> data = jsonDecode(response.body);
  final list = data.map((e) => Country.fromJson(e)).toList();
  list.sort((a, b) => a.name.compareTo(b.name));
  return list;
}
```

### 3. Search and Region Filter
**File:** `lib/screens/home_screen.dart`

```dart
List<Country> get filtered {
  return allCountries.where((c) {
    final matchesName =
        c.name.toLowerCase().contains(query.trim().toLowerCase());
    final matchesRegion =
        selectedRegion == 'All' || c.region == selectedRegion;
    return matchesName && matchesRegion;
  }).toList();
}
```

### 4. Loading Data and Handling Errors
**File:** `lib/screens/home_screen.dart`

```dart
Future<void> loadCountries() async {
  setState(() {
    isLoading = true;
    error = '';
  });

  try {
    final result = await widget.fetcher();
    setState(() {
      allCountries = result;
      isLoading = false;
    });
  } catch (e) {
    setState(() {
      error = 'Could not load countries. Check your connection.';
      isLoading = false;
    });
  }
}
```

### 5. Search Bar and Region Chips
**File:** `lib/screens/home_screen.dart`

```dart
TextField(
  key: const Key('searchField'),
  decoration: const InputDecoration(
    prefixIcon: Icon(Icons.search),
    hintText: 'Search country',
  ),
  onChanged: (value) => setState(() => query = value),
),

Wrap(
  spacing: 8,
  children: regions.map((r) => ChoiceChip(
    key: Key('chip_$r'),
    label: Text(r),
    selected: selectedRegion == r,
    onSelected: (_) => setState(() => selectedRegion = r),
  )).toList(),
),
```

### 6. Country Card
**File:** `lib/screens/home_screen.dart`

```dart
Card(
  child: ListTile(
    leading: Image.network(
      c.flag,
      width: 56,
      errorBuilder: (_, __, ___) => const Icon(Icons.flag),
    ),
    title: Text(c.name),
    subtitle: Text(
      '${c.region} • Pop: ${formatPopulation(c.population)}\n'
      '${c.languages.join(", ")}',
    ),
    onTap: () => Navigator.push(context, MaterialPageRoute(
      builder: (_) => CountryDetailScreen(country: c),
    )),
  ),
)
```

### 7. Widget Test Code
**File:** `test/widget_test.dart`

```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import '../lib/main.dart';
import '../lib/models/country.dart';

Future<List<Country>> mockFetch() async => [
  Country(code: 'IND', name: 'India', flag: '', region: 'Asia',
      capital: 'New Delhi', population: 1428627663,
      languages: ['Hindi', 'English']),
  Country(code: 'FRA', name: 'France', flag: '', region: 'Europe',
      capital: 'Paris', population: 67391582, languages: ['French']),
];

void main() {
  testWidgets('Test 3: Search filters the list', (tester) async {
    await tester.pumpWidget(CountryExplorerApp(fetcher: mockFetch));
    await tester.pumpAndSettle();

    await tester.enterText(find.byKey(const Key('searchField')), 'fra');
    await tester.pumpAndSettle();

    expect(find.text('France'), findsOneWidget);
    expect(find.text('India'), findsNothing);
  });

  // Add the remaining tests (Test 1, 2, 4, 5, 6, 7) here
}
```

> 📁 The complete source code is available in the `lib/` and `test/` folders of this repository.

## 🧪 Widget Testing

Flutter's `flutter_test` package is used to verify the functionality and reliability of the application. All tests use a **mocked fetch function**, so they run fast and do not need internet access.

### Test Cases

| Test Case | Description |
|---|---|
| Test 1 | Initial load – country cards appear after the mock data loads |
| Test 2 | Card content – name, region, population and languages are shown |
| Test 3 | Search – typing "fra" shows France and hides India |
| Test 4 | Region filter – tapping the Asia chip shows only Asian countries |
| Test 5 | Search + filter combined – both conditions apply together |
| Test 6 | Empty result – "No countries found" appears for a name that does not exist |
| Test 7 | Error state – error message and Retry button appear when the fetch fails |

### Testing Methods Used

- `testWidgets()`
- `pumpWidget()`
- `enterText()`
- `tap()`
- `pumpAndSettle()`
- `find.byKey()`
- `find.text()`
- `expect()`

### Run the Tests

```bash
flutter test
```

### Expected Test Output

```
All 7 widget tests passed!
```

## 🛠️ Technologies Used

- Flutter
- Dart
- Material 3
- `http` package
- `dart:convert`
- Flutter Test
- Visual Studio Code

## 📂 Project Structure

```
country_explorer/
│
├── lib/
│   ├── main.dart
│   ├── models/
│   │   └── country.dart
│   ├── services/
│   │   └── country_service.dart
│   └── screens/
│       ├── home_screen.dart
│       └── detail_screen.dart
│
├── test/
│   └── widget_test.dart
│
├── screenshots/
├── android/
├── web/
├── pubspec.yaml
├── analysis_options.yaml
└── README.md
```

## ▶️ How to Run

```bash
git clone <your-repository-link>
cd country_explorer
flutter pub get
flutter run
```

## 🌍 SDG Relevance

**SDG 4 – Quality Education**

This project makes geography and world knowledge easy to access. Learners can quickly compare populations, discover languages and explore regions, supporting self-directed learning and global awareness. Using an open API also shows how open data can support education.

Automated widget testing improves application reliability by verifying search, filtering, state handling and result display.

> **Note:** Population and country data are provided by the public REST Countries API and may change over time.

## 🎯 Learning Outcomes

- Consuming a REST API in Flutter using the `http` package and `async` / `await`.
- Parsing JSON into typed Dart model classes.
- Using `StatefulWidget` and `setState()` to manage search and filter state.
- Building list UIs with `ListView.builder`, `Card`, `ListTile`, `ChoiceChip` and `TextField`.
- Handling loading, error and empty states.
- Navigating between screens using `Navigator.push`.
- Writing widget tests with dependency injection and mock data.
- Verifying application functionality using test assertions.

## 👨‍🎓 Project Details

- **College:** Vishnu Institute of Technology :: Bhimavaram
- **Department:** Computer Science and Engineering (CSE) – Section D
- **Course:** User Interface Design Using Flutter
- **Course Instructor:** Mrs. P. Shyamala Madhuri
- **Head of Department:** Dr. R. Srinivasa Raju
