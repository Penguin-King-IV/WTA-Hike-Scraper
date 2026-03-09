# WTA Hike Scraper

Scrapes hike data from [Washington Trails Association](https://www.wta.org) and stores it in a filterable Java `HikeRepository`.

I did not make this it was made by Claude.ai chat link: (https://claude.ai/share/c764f005-12d4-4458-be30-8b6c50b62201)

For Installation and usage use The Washington Trail Assosation Scraper Manual.

## Data Collected Per Hike

| Field | Type | Example |
|-------|------|---------|
| `name` | String | "Lake Serene" |
| `areaLocation` | String | "Index Area" |
| `lengthMiles` | double | 8.0 |
| `highestPointFeet` | double | 5,200 |
| `elevationGainFeet` | double | 2,200 |
| `tags` | List\<String\> | ["Dogs allowed on leash", "Wildflowers/Meadows"] |
| `url` | String | "https://www.wta.org/go-hiking/hikes/lake-serene" |

---

## Requirements

- Java 17+
- Maven 3.8+

---

## Build & Run

```bash
# Build the fat jar
mvn clean package -q

# Scrape 60 hikes (default, 2 listing pages)
java -jar target/wta-scraper.jar

# Scrape 300 hikes
java -jar target/wta-scraper.jar 300

# Scrape everything (will take a while — ~8,000+ hikes on WTA)
java -jar target/wta-scraper.jar 99999
```

Output:
- `hikes.json` — all scraped hikes as JSON (can be reloaded without re-scraping)
- Console — filtered results from `Main.java` examples

---

## Using HikeRepository (Java Streams)

```java
HikeRepository repo = new HikeRepository(allHikes);

// ── Chainable filters ──────────────────────────────────────────────────────

// Dog-friendly day hikes in the Olympics
List<Hike> results = repo
    .filterByArea("Olympics")
    .filterByTag("Dogs allowed on leash")
    .filterByMaxLength(10)
    .sortByElevationGain()
    .toList();

// Wildflower hikes above 5,000 ft
List<Hike> highWildflowers = repo
    .filterByTag("Wildflowers/Meadows")
    .filterByMinElevation(5_000)
    .sortByHighestPoint()
    .toList();

// Kid-friendly hikes under 4 miles with low gain
List<Hike> easyKidHikes = repo
    .filterByAllTags("Kids", "Good for kids")
    .filterByMaxLength(4)
    .filterByMaxGain(500)
    .sortByLength()
    .toList();

// ── Aggregations ───────────────────────────────────────────────────────────

repo.hardestHike();          // Optional<Hike> — most elevation gain
repo.longestHike();          // Optional<Hike> — longest distance
repo.averageLength();        // double
repo.groupByArea();          // Map<String, List<Hike>>
repo.allTags();              // List<String> — every unique tag

// ── Raw stream access ──────────────────────────────────────────────────────

long snowshoeCount = repo.stream()
    .filter(h -> h.hasTag("Snowshoeing"))
    .count();
```

---

## Available WTA Tags (common)

- `Dogs allowed on leash`
- `Dogs not allowed`
- `Wildflowers/Meadows`
- `Mountain views`
- `Summits`
- `Old growth`
- `Lakes`
- `Rivers`
- `Waterfalls`
- `Wildlife`
- `Established campsites`
- `Good for kids`
- `Snowy most of year`
- `Snowshoeing`

Run `repo.allTags()` after scraping to get the full current list.

---

## Reloading from JSON (skip re-scraping)

```java
ObjectMapper mapper = new ObjectMapper();
List<Hike> hikes = mapper.readValue(
    new File("hikes.json"),
    mapper.getTypeFactory().constructCollectionType(List.class, Hike.class)
);
HikeRepository repo = new HikeRepository(hikes);
```

---

## Project Structure

```
wta-scraper/
├── pom.xml
└── src/main/java/com/wta/
    ├── model/
    │   └── Hike.java              ← data model
    └── scraper/
        ├── WtaScraper.java        ← HTTP + HTML parsing (Jsoup)
        ├── HikeRepository.java    ← in-memory store + Stream filters
        └── Main.java              ← entry point + usage examples
```
