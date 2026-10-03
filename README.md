
# Optimizing the Mjengo CMS Material Usage Dashboard

### A practical performance improvement

**Focus:** database aggregation, interface projections, and pagination  
**System:** Mjengo CMS construction management platform

---

## The dashboard requirement

The dashboard needed to show:

- The most-used construction materials
- For a specific construction site
- Within a selected date range
- Ordered from highest to lowest usage

Example:

```text
Cement       450 bags
Sand         320 tonnes
Steel        180 pieces
```

---

## The initial approach

The application first fetched many raw material-use records.

Then Java had to:

1. Filter by construction site
2. Filter by date range
3. Group records by material
4. Add the quantities
5. Sort the totals
6. Select the top results

```text
Database → raw records → Java processing → dashboard
```

---

## Why this became a performance problem

The application was doing work that the database is designed to do.

- Too many records transferred to the application
- More Java objects created in memory
- More CPU and memory usage
- Slower dashboard responses
- Poorer scalability as daily logs increased

The dashboard only needed a small summary, not every raw record.

---

## The optimized query

```java
@Query("""
    SELECT m.materialName AS materialName,
           SUM(m.quantityConsumed) AS totalAmount
    FROM DailyLog d
    JOIN d.materialsUsed m
    WHERE d.constructionId = :constructionId
      AND d.logDate BETWEEN :startDate AND :endDate
    GROUP BY m.materialName
    ORDER BY totalAmount DESC
""")
List<DashboardProjections.MaterialUsageProjection>
findTopUsedMaterials(
    String constructionId,
    LocalDate startDate,
    LocalDate endDate,
    Pageable pageable
);
```

---

## What the database now does

The database performs the complete data operation:

1. Filters by construction site
2. Filters by date range
3. Joins daily logs with used materials
4. Groups by material name
5. Calculates totals with `SUM`
6. Sorts by total usage
7. Limits results through `Pageable`

```text
Database processes data → application receives final summary
```

---

## Interface-based projection

```java
public interface MaterialUsageProjection {
    String getMaterialName();
    BigDecimal getTotalAmount();
}
```

The query returns only the fields the dashboard needs:

- `materialName`
- `totalAmount`

Spring Data maps the query aliases directly to the interface getters.

No complete entity objects are required.

---

## Before and after

### Before

```text
Fetch raw records
→ transfer many rows
→ create entities
→ filter and aggregate in Java
→ return top results
```

### After

```text
Database filters, joins, groups, sums, sorts, and limits
→ projection returns two fields
→ dashboard receives a small result
```

---

## Result and key lesson

### Improvements

- Less data transferred
- Lower memory usage
- Less application-side processing
- Faster dashboard response
- Better scalability

### Key lesson

> Performance is not only about writing faster application code. It is also about sending less data to the application and allowing each layer to do the work it is best suited for.

**Database:** data processing  
**Application:** business logic and presentation

---

# Thank you

## Questions?

**Challenge:** Optimizing dashboard material-usage queries  
**Technologies:** Spring Boot · Spring Data JPA · JPQL · MySQL
