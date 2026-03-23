# EXAMEN · MÒDUL 489

## Programació de Dispositius Mòbils i Multimèdia

**Unitats Formatives:** RA2 i RA3  
**Curs:** 2n DAM · Videojocs  
**Data:** 23/03/26 
**Durada:** 2 hores  

**Alumne/a:** Edim Batalla  
**Grup:** 2n DAM

---

## Posada en marxa de l'entorn

Consulta el fitxer **`README.md`** del projecte per a les instruccions completes d'instal·lació i arrencada (Node.js, servidor mock i Flutter).

---

> **Instruccions generals**
>
> - Respon cada pregunta en l'espai indicat (substitueix el text `[Escriu la teva resposta aquí]`).
> - Per a la part de codi, escriu directament en bloc `dart`. No és necessari que el codi compili, però ha de reflectir coneixement real de Flutter/Dart.
> - Tens el codi dels projectes **Cars** i **Phone** com a referència en el teu ordinador. **No pots accedir a internet** durant l'examen.
> - Desa el fitxer i lliura'l amb el nom: `EXAMEN_M489_[el_teu_nom].md`
> - Fes commit i push del .md modificat i de tots els arxius que hagis modificat

---

## BLOC 1 · ARQUITECTURA I CICLE DE VIDA *(RA 2)*

### Pregunta 1.1 – Comunicació entre Widgets *(12 punts)*

Al projecte **Cars**, el widget `CarsPage` gestiona el número de pàgina actual (`_currentPage`) i el passa a `CarsList1`. El widget `ButtonPanel` conté els botons "Anterior" i "Següent".

**a)** A `cars_page.dart`, el widget utilitza el mètode `setState` per gestionar la paginació. Explica:

- Quina és la funció de `setState` i per què cridar-lo fa que la UI es torni a dibuixar.
- Per quin motiu `_loadPage()` fa servir dos crides a `setState` separades (una a l'inici i una al final) en lloc d'una sola.

**Resposta:**

```
setState() serveix per indicar que l’estat del widget ha canviat i que cal tornar a executar el mètode build per tornar a construir la interfície amb nous valors. 

_loadPage() fa 2 crides separades perquè actualitza dos moments diferents l’estat:
A l’inici per posar loading = true i mostrar a la UI que s’estan carregant dades.
I al final per guardar la nova llista o el número de pàgina.
```

---

### Pregunta 1.2 – Cicle de vida d'un widget amb recursos *(13 punts)*

Al projecte **Camera**, el widget `CameraScreen` utilitza un `CameraController` per gestionar la càmera del dispositiu. Aquest controlador ocupa recursos del sistema (càmera, memòria) i cal alliberar-los correctament.

**a)** Quin mètode del cicle de vida de `State` s'usa a `CameraScreen` per alliberar el `CameraController` quan el widget és destruït? Escriu com es fa i explica per quina raó és imprescindible cridar-lo.

**Resposta:**

```
S’utilitza dispose(). i es crida _controller.dispose() per alliberar la càmera i la memòria.
```

---

**b)** El `CameraController` s'inicialitza de forma asíncrona a `initState()` i el resultat es guarda a `_initializeControllerFuture`. Respon les preguntes següents:

- Per quin motiu no es pot fer `await` directament a `initState()`?
- Quina millora aporta a l'usuari usar `FutureBuilder` en lloc de bloquejar el fil?
- Com treballen junts `_initializeControllerFuture` i `FutureBuilder`?

**Resposta:**

```
No es pot fer await directament a initState() perquè aquest mètode ha de ser síncron. El Future es guarda a _initializeControllerFuture i FutureBuilder el fa servir per saber en quin estat està.
```

---

## BLOC 2 · COMUNICACIÓ, PERSISTÈNCIA I PROVES *(RA 2 — 35 punts)*

### Pregunta 2.1 – Consum d'API i robustesa *(18 punts)*

Analitza el mètode `getCarsPage(int page, int limit)` de `car_http_service.dart`.

Què passaria si el servidor de l'API trigués 60 segons a respondre? L'aplicació quedaria bloquejada per a l'usuari? Per què? Escriu com implementaries un *timeout* de 10 segons a la petició HTTP.

**Resposta:**

Si el servidor triga molt l’app no es bloqueja perquè la crida és asíncrona però l’usuari es queda esperant. Per això s'utilitza timeout de 10 segons per exemple.

```dart
Future<List<CarsModel>> getCarsPage(int page, int limit) async {
  try {
     final uri = _buildUri('/v1/cars', {'limit': '$limit', 'offset': '$offset'});


    final response = await http
        .get(uri)
        .timeout(const Duration(seconds: 10));

    if (response.statusCode !=200) {
      throw Exception(
        'Error ${response.statusCode}',
      );
    }
}
```

---

### Pregunta 2.2 – Models de dades  *(17 punts)*

Analitza el constructor `factory CarsModel.fromMapToCarObject(Map<String, dynamic> json)` de `car_model.dart`.

**a)** Imagina que l'API retorna per error el camp `year` com a `String` en lloc d'`int` (per exemple, `"2021"` en lloc de `2021`). El codi actual fallaria. Escriu com resoldries el problema.

**Resposta:**

```
Cal convertir year  amb int.tryParse(...). Això evita errors si l’API envia un String en lloc d’un int.
```

---

**b)** Al fitxer `class_model_test.dart`, el test utilitza un `const jsonString` amb un JSON escrit a mà en lloc de fer una petició real a l'API de RapidAPI. Explica per quin motiu és millor simular el JSON en un test unitari.

**Resposta:**

```
En un test unitari és millor simular el JSON perquè així el test no depèn d’internet ni de l’API. 
```

---

## BLOC 3 · IMPLEMENTACIÓ PRÀCTICA *(RA 3 — 30 punts)*

### Exercici – Widget de detall amb dades remotes

Imagina que volem crear una pantalla de detall per a cada cotxe del projecte Cars. Implementa el mètode `build` d'un widget `StatelessWidget` anomenat `CarDetailPage` que compleixi els requisits següents:

1. Rep un paràmetre `final CarsModel car` al constructor.
2. Mostra el **make** i el **model** del cotxe com a títol destacat (`Text` amb estil gran i negreta).
3. Mostra una **icona diferent** depenent del `type` del cotxe:
   - Si el `type` és `'SUV'`, mostra `Icons.directions_car`.
   - Per qualsevol altre tipus, mostra `Icons.car_rental`.
4. Afegeix un botó `ElevatedButton` que, quan es premi, mostri un `SnackBar` amb el text: `"Cotxe seleccionat: [make] [model]"`.

```dart
class CarDetailPage extends StatelessWidget {
  final CarsModel car;

  const CarDetailPage({super.key, required this.car});

  @override
  Widget build(BuildContext context) {
    final IconData carIcon = car.type == 'SUV'
        ? Icons.directions_car
        : Icons.car_rental;

    return Scaffold(
      appBar: AppBar(
        title: const Text('Detall del cotxe'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              '${car.make} ${car.model}',
              style: const TextStyle(
                fontSize: 24,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 16),
            Icon(
              carIcon,
              size: 48,
            ),
            const SizedBox(height: 24),
            ElevatedButton(
              onPressed: () {
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(
                    content: Text(
                      'Cotxe seleccionat: ${car.make} ${car.model}',
                    ),
                  ),
                );
              },
              child: const Text('Seleccionar cotxe'),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

**Ampliació (nivell Expert):** Afegeix un `FutureBuilder` que cridi al mètode `CarHttpService().getCarsPage(1, 5)` i mentre espera les dades mostri un `CircularProgressIndicator`. Quan les dades estiguin llestes, mostra un `ListView.builder` amb el `make` de cada cotxe. Si hi hagués un error, mostra un `Text` en color vermell amb el missatge de l'error.

```dart
//Escriu la teva ampliació aquí:
```

---

## BLOC 4 · EXTENSIÓ DEL SERVEI HTTP *(RA 2 — 10 punts)*

### Exercici 4.1 – Mètode parametritzat a `CarHttpService` *(10 punts)*

El servidor mock local té disponible un  endpoint de cerca:

```
GET http://localhost:8080/v1/cars/search?make=Toyota&model=Corolla
```

- El paràmetre `make` filtra per marca (coincidència parcial, insensible a majúscules).
- El paràmetre `model` filtra per model (coincidència parcial, insensible a majúscules).
- Tots dos paràmetres són opcionals: si no s'envien, retorna tots els cotxes.

Exemples vàlids:

- `/v1/cars/search?make=Toyota` → tots els Toyota
- `/v1/cars/search?model=X5` → tots els cotxes amb "X5" al model
- `/v1/cars/search?make=BMW&model=X` → BMW amb "X" al model

**Implementa** el mètode `getCarsByFilter` a la classe `CarHttpService` existent, seguint el mateix patrons que `getCarsPage`:

```dart
Future<List<CarsModel>> getCarsByFilter({
  String? make,
  String? model,
}) async {
  try {
    final queryParams = <String, String> {};

    if (make != null && make.isNotEmpty) {
      queryParams['make'] = make;
    }

    if (model != null && model.isNotEmpty) {
      queryParams['model'] = model;
    }

    final uri = _buildUri('/v1/cars/search', queryParams);

    final response = await http
        .get(uri, headers: -headers)
        .timeout(const Duration(seconds: 10));

    if (response.statusCode == 200) {
      final List<dynamic> jsonList = jsonDecode(response.body);
      return jsonList
          .map((json) => CarsModel.fromMapToCarObject(json))
          .toList();
    }
  }
}
```

Requisits:

1. Utilitza el mètode privat `_buildUri(String path, Map<String, String> queryParams)` ja existent.
2. Només afegeix els paràmetres `make` i/o `model` al mapa si el valor no és `null` ni buit (`isEmpty`).
3. Gestiona errors i timeout amb el mateix mecanisme que `getCarsPage`.

**Resposta:**

```dart

// Escriu aquí la teva implementació completa del mètode:

```

---

## Resum de l'examen

| Bloc | RA | Punts màxims |
|------|----|:------------:|
| Bloc 1 – Arquitectura i Cicle de vida | RA 2 | 25 |
| Bloc 2 – Comunicació, Persistència i Proves | RA 2  | 35 |
| Bloc 3 – `CarDetailPage` (base) | RA 3 | 20 |
| Bloc 3 – Ampliació `FutureBuilder`  | RA 3 | 10 |
| Bloc 4 – Extensió del servei HTTP | RA 2 | 10 |
| **TOTAL** | | **100** |

---