# Design Document

## 1) Components

- **Animal Data**

  - **Animal Attribute**
    - Extracts animal attributes (e.g., ages, gender, species) from `arrivingAnimals.txt` for respective species.

  - **Name Pool**
    - Extracts animal names from `animalNames.txt` to respective species.

- **Animal Types**
  - Set subclasses and each subclasses has objectID for Animal Data.

- **Input (Scanner)**
  - Reads text files to get animal data.

- **IO Error**
  - Captures errors for the flaw in code.

- **Output Report (PrintWriter)**
  - Creates a text file (`newAnimals.txt`) of animals and their attributes sorted in category.

## 2) Data structures

- `Map<String, List<String>> namePool`
  - Get name depending on species

- `Map<String, List<Animal>> sortSpecies`
  - Sort habitats

## 3) Interactions

1. Text file get read via filepath.
2. Import animal names into namePool.
3. Pull animal's attribute from `arrivingAnimals.txt`.
4. Sort by Species > Animal ID > Attributes and repeat.
5. Print out the report.

# Code Review

## From main.java

```java
// Old version

private static void loadArrivals(String file) throws IOException {
        try (Scanner sc = new Scanner(new File(file))) {
            while (sc.hasNextLine()) {
                String line = sc.nextLine().toLowerCase();
                String[] parts = line.split(","); // Separate words into indexes.
                String[] tokens = parts[0].split(" "); // tokens: [age, "year", "old", gender, species, ...]
                int age = Integer.parseInt(tokens[0]);
                String species = tokens[tokens.length - 1]; // Get species token.
                String speciesKey = capitalize(species);

                List<String> pool = namePool.getOrDefault(speciesKey, Collections.emptyList()); // Fetch names from namePool to assign animals with a name.
                String name = pool.isEmpty() ? "Unnamed" : pool.get(new Random().nextInt(pool.size()));

                Animal a; // Creates animals
                switch (species) {
                    case "hyena":
                        a = new Hyena(name, age);
                        ((Hyena) a).setPattern("Striped");
                        break;
                    case "lion":
                        a = new Lion(name, age);
                        ((Lion) a).setHabitat("Grasslands");
                        break;
                    case "bear":
                        a = new Bear(name, age);
                        ((Bear) a).setFurColor("Brown");
                        break;
                    case "tiger":
                        a = new Tiger(name, age);
                        ((Tiger) a).setStripePattern("Bold");
                        break;
                    default:
                        continue;
                }
                sortSpecies.computeIfAbsent(speciesKey, k -> new ArrayList<>()).add(a); // Store and sort species into respective category.
            }
        }
    }
```

```java
// Improvement; better readability because I turned the big function to smaller pieces.

private static void loadArrivals(String filePath) throws IOException {
    try (Scanner scanner = new Scanner(new File(filePath))) {
        while (scanner.hasNextLine()) {
            String line = scanner.nextLine().toLowerCase().trim();
            if (line.isEmpty()) continue;

            Animal animal = parseAnimalLine(line);
            if (animal != null) {
                addToSpeciesList(animal);
            }
        }
    }
}

private static Animal parseAnimalLine(String line) {
    String[] parts = line.split(",");
    if (parts.length < 1) return null;

    String[] tokens = parts[0].split("\\s+");
    if (tokens.length < 4) return null;

    int age = Integer.parseInt(tokens[0]);
    String species = tokens[tokens.length - 1];
    String speciesKey = capitalize(species);

    String name = getRandomName(speciesKey);

    return createAnimal(species.toLowerCase(), name, age);
}

private static String getRandomName(String species) {
    List<String> pool = namePool.getOrDefault(species, Collections.emptyList());
    return pool.isEmpty() ? "Unnamed" : 
            pool.get(new Random().nextInt(pool.size()));
}

private static Animal createAnimal(String species, String name, int age) {
    return switch (species) {
        case "hyena" -> {
            Hyena h = new Hyena(name, age);
            h.setPattern("Striped");
            yield h;
        }
        case "lion" -> {
            Lion l = new Lion(name, age);
            l.setHabitat("Grasslands");
            yield l;
        }
        case "bear" -> {
            Bear b = new Bear(name, age);
            b.setFurColor("Brown");
            yield b;
        }
        case "tiger" -> {
            Tiger t = new Tiger(name, age);
            t.setStripePattern("Bold");
            yield t;
        }
        default -> null;
    };
}
```

# Experiment

```java
private static Animal parseAnimalLine(String line) {
    // ...
    String color = "Unknown";
    for (int i = 1; i < parts.length; i++) {
        String segment = parts[i].trim().toLowerCase();
        if (segment.contains("color")) {
            color = segment.replace("color", "").trim();
            break;
        }
    }
}
```