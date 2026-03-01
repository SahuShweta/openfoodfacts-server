# Testing & Validation Guide for Issue #13117

## 🧪 Pre-Merge Testing

All examples in this PR can be tested immediately against the live API.

---

## Test Categories

### 1️⃣ Pagination Testing

#### Test Case 1.1: Default Pagination
```bash
curl -s "https://world.openfoodfacts.org/api/v2/search?fields=code,product_name&page_size=5" | jq '.page, .page_size, .page_count, .count'
```

**Expected Output:**
```json
1
5
(large number - total pages)
(large number - total products)
```

**✓ Validates:** Default page (1) works, page_size respected, response fields present

---

#### Test Case 1.2: Pagination Boundaries
```bash
# Test page 2
curl -s "https://world.openfoodfacts.org/api/v2/search?page=2&page_size=10" | jq '.page, .skip, (.products | length)'

# Expected: page=2, skip=10, products.length=10
```

**Validates:** page parameter, skip offset calculation

---

#### Test Case 1.3: Invalid Page Number
```bash
curl -s "https://world.openfoodfacts.org/api/v2/search?page=999999&page_size=10" | jq '.products | length'
```

**Expected:** `0` (empty products array, but count still shows total)

**Validates:** Edge case handling documented

---

### 2️⃣ Tag Filter Testing

#### Test Case 2.1: Single Tag Filter
```bash
curl -s "https://world.openfoodfacts.org/api/v2/search?nutrition_grades_tags=a&page_size=5&fields=code,nutrition_grades" | jq '.products[0].nutrition_grades'
```

**Expected:** All products have nutrition_grades="a"

**Validates:** Tag filtering works

---

#### Test Case 2.2: AND Operator (Comma-Separated)
```bash
# Both organic AND fair-trade
curl -s "https://world.openfoodfacts.org/api/v2/search?labels_tags=en:organic,en:fair-trade&page_size=10" | jq '.count'
```

Compare count with single filter:
```bash
curl -s "https://world.openfoodfacts.org/api/v2/search?labels_tags=en:organic&page_size=10" | jq '.count'
```

**Expected:** AND count < OR count (more restrictive)

**Validates:** AND operator (comma) works

---

#### Test Case 2.3: OR Operator (Pipe-Separated)
```bash
# Either grade A or grade B
curl -s "https://world.openfoodfacts.org/api/v2/search?nutrition_grades_tags=a|b&page_size=5&fields=code,nutrition_grades" | jq '.products[].nutrition_grades'
```

**Expected:** All results have grades "a" OR "b"

**Validates:** OR operator (pipe) works

---

#### Test Case 2.4: NOT Operator (Minus Prefix)
```bash
# Organic but NOT fair-trade
curl -s "https://world.openfoodfacts.org/api/v2/search?labels_tags=en:organic,-en:fair-trade&page_size=10" | jq '.count'
```

Compare with just organic:
```bash
curl -s "https://world.openfoodfacts.org/api/v2/search?labels_tags=en:organic&page_size=10" | jq '.count'
```

**Expected:** NOT count < organic count (more restrictive)

**Validates:** NOT operator (minus) works

---

### 3️⃣ Nutrient Filter Testing

#### Test Case 3.1: Less Than Operator
```bash
curl -s "https://world.openfoodfacts.org/api/v2/search?sugars_100g<5&page_size=5&fields=code,product_name,sugars_100g" | jq '.products[].sugars_100g'
```

**Expected:** All values < 5 (or missing if not recorded)

**Validates:** `<` operator works

---

#### Test Case 3.2: Greater Than Operator
```bash
curl -s "https://world.openfoodfacts.org/api/v2/search?protein_100g>10&page_size=5&fields=code,product_name,protein_100g" | jq '.products[].protein_100g'
```

**Expected:** All values > 10 (or missing)

**Validates:** `>` operator works

---

#### Test Case 3.3: Exact Match Operator
```bash
curl -s "https://world.openfoodfacts.org/api/v2/search?energy-kj_100g=2000&page_size=5&fields=code,energy-kj_100g" | jq '.products[].energy-kj_100g'
```

**Expected:** All values = 2000

**Validates:** `=` operator works

---

#### Test Case 3.4: Multiple Nutrient Filters (AND)
```bash
# Low sugar AND high protein
curl -s "https://world.openfoodfacts.org/api/v2/search?sugars_100g<10&protein_100g>5&page_size=5&fields=code,sugars_100g,protein_100g" | jq '.products[] | select(.sugars_100g != null and .protein_100g != null)'
```

**Expected:** All products have low sugar AND high protein

**Validates:** Multiple nutrient filters work together

---

### 4️⃣ Sorting Testing

#### Test Case 4.1: Sort by Product Name
```bash
curl -s "https://world.openfoodfacts.org/api/v2/search?sort_by=product_name&page_size=5&fields=code,product_name" | jq '.products[].product_name'
```

**Expected:** Products in alphabetical order (A-Z)

**Validates:** `sort_by=product_name` works

---

#### Test Case 4.2: Sort by Nutri-Score
```bash
curl -s "https://world.openfoodfacts.org/api/v2/search?sort_by=nutriscore_score&page_size=5&fields=code,nutrition_grades" | jq '.products[].nutrition_grades'
```

**Expected:** Grades appear in order A, B, C, D, E (best to worst)

**Validates:** `sort_by=nutriscore_score` works

---

#### Test Case 4.3: Sort by Last Modified
```bash
curl -s "https://world.openfoodfacts.org/api/v2/search?sort_by=last_modified_t&page_size=3&fields=code,last_modified_t" | jq '.products[].last_modified_t'
```

**Expected:** Timestamps in descending order (most recent first)

**Validates:** `sort_by=last_modified_t` works

---

### 5️⃣ Response Limiting Testing

#### Test Case 5.1: Fields Parameter Reduces Size
```bash
# Full response size
curl -s "https://world.openfoodfacts.org/api/v2/search?page_size=1" | wc -c

# Limited fields
curl -s "https://world.openfoodfacts.org/api/v2/search?page_size=1&fields=code,product_name" | wc -c
```

**Expected:** Limited response significantly smaller than full

**Validates:** `fields` parameter reduces payload as documented

---

#### Test Case 5.2: Only Requested Fields Returned
```bash
curl -s "https://world.openfoodfacts.org/api/v2/search?page_size=1&fields=code,product_name" | jq '.products[0] | keys'
```

**Expected:** Only `code` and `product_name` fields (plus response metadata)

**Validates:** Response limiting works correctly

---

### 6️⃣ Complex Real-World Tests

#### Test Case 6.1: Healthy Beverages (From Docs)
```bash
curl -s "https://world.openfoodfacts.org/api/v2/search?categories_tags_en=non-alcoholic-beverages&nutrition_grades_tags=a|b&sugars_100g<5&fields=code,product_name,nutrition_grades,sugars_100g&page_size=3" | jq '.count'
```

**Expected:** count > 0 (results found)

**Validates:** Complex multi-filter query works

---

#### Test Case 6.2: Organic + Fair-Trade (From Docs)
```bash
curl -s "https://world.openfoodfacts.org/api/v2/search?labels_tags=en:organic,en:fair-trade&sort_by=product_name&fields=code,product_name,brands&page=1&page_size=5" | jq '.products | length'
```

**Expected:** 5 results (or less if not available)

**Validates:** Multi-label AND filtering works

---

#### Test Case 6.3: Low-Sodium Snacks (From Docs)
```bash
curl -s "https://world.openfoodfacts.org/api/v2/search?categories_tags_en=snacks&salt_100g<0.3&sort_by=nutriscore_score&fields=code,product_name,salt_100g,nutrition_grades&page_size=3" | jq '.products | length > 0'
```

**Expected:** true (results found)

**Validates:** Snack category + nutrient filter works

---

## 📋 Test Execution Checklist

Run all tests with this script:

```bash
#!/bin/bash

echo "=== Test 1.1: Default Pagination ==="
curl -s "https://world.openfoodfacts.org/api/v2/search?fields=code&page_size=5" | jq '.page'

echo "=== Test 2.1: Single Tag Filter ==="
curl -s "https://world.openfoodfacts.org/api/v2/search?nutrition_grades_tags=a&page_size=3" | jq '.count'

echo "=== Test 2.3: OR Operator ==="
curl -s "https://world.openfoodfacts.org/api/v2/search?nutrition_grades_tags=a|b&page_size=3" | jq '.count'

echo "=== Test 3.1: Nutrient < Filter ==="
curl -s "https://world.openfoodfacts.org/api/v2/search?sugars_100g<5&page_size=3" | jq '.count'

echo "=== Test 4.3: Sort by Last Modified ==="
curl -s "https://world.openfoodfacts.org/api/v2/search?sort_by=last_modified_t&page_size=1&fields=code" | jq '.products | length'

echo "=== Test 6.1: Complex Query ==="
curl -s "https://world.openfoodfacts.org/api/v2/search?categories_tags_en=beverages&nutrition_grades_tags=a|b&sugars_100g<5&page_size=3" | jq '.count'

echo "All tests completed!"
```

---

## ✅ Acceptance Criteria

All tests should pass:
- [ ] Pagination defaults work (page=1, page_size=24)
- [ ] All tag filters return results
- [ ] AND operator (comma) is more restrictive than single filter
- [ ] OR operator (pipe) returns more results than single filter
- [ ] NOT operator (minus) reduces results
- [ ] Nutrient operators (<, >, =) work correctly
- [ ] All 11 sort options produce valid responses
- [ ] fields parameter reduces response size
- [ ] Complex queries with multiple filters work
- [ ] All 4 documentation examples return results

---

## 🐛 If Tests Fail

| Failure | Diagnosis |
|---------|-----------|
| Pagination returns page number different from requested | API might queue requests differently - check skip value |
| Tag filter returns empty results | Tag might not exist in database - try known tag like `nutrition_grades_tags=a` |
| Nutrient filter returns no results | Nutrient data might not be recorded for that category - try broader category |
| Response has unexpected fields despite `fields` param | Some fields might be added by middleware - check if optional |
| Sort doesn't appear to work | Might need multiple pages to see sorting effect |

---

## 📊 Performance Testing (Optional)

To ensure documentation recommendations work:

```bash
# Test with fields parameter (RECOMMENDED)
time curl -s "https://world.openfoodfacts.org/api/v2/search?page_size=100&fields=code,product_name" > /dev/null

# Test without fields parameter (NOT RECOMMENDED)
time curl -s "https://world.openfoodfacts.org/api/v2/search?page_size=100" > /dev/null
```

**Expected:** First request should be noticeably faster

---

## 🔍 Documentation Validation

After testing API functionality:

1. ✅ Check `search-parameters.md` renders correctly in docs site
2. ✅ Verify all internal links work (relative paths)
3. ✅ Confirm code examples are properly formatted
4. ✅ Validate all parameter names match API spec
5. ✅ Check that language suffix examples are correct
6. ✅ Confirm operator examples are accurate

---

## 📝 Test Report Template

```markdown
# Test Results for PR #13117

**Date:** [Date]
**Tester:** [Name]
**API Environment:** Production (world.openfoodfacts.org)

## Tests Run: 19/19 ✅

### Pagination (3 tests)
- [x] Test 1.1: Default pagination
- [x] Test 1.2: Pagination boundaries
- [x] Test 1.3: Invalid page number

### Tag Filters (4 tests)
- [x] Test 2.1: Single tag
- [x] Test 2.2: AND operator
- [x] Test 2.3: OR operator
- [x] Test 2.4: NOT operator

### Nutrient Filters (4 tests)
- [x] Test 3.1: Less than
- [x] Test 3.2: Greater than
- [x] Test 3.3: Exact match
- [x] Test 3.4: Multiple filters

### Sorting (3 tests)
- [x] Test 4.1: Sort by name
- [x] Test 4.2: Sort by score
- [x] Test 4.3: Sort by date

### Response Limiting (2 tests)
- [x] Test 5.1: Fields reduces size
- [x] Test 5.2: Only requested fields returned

### Complex Queries (3 tests)
- [x] Test 6.1: Healthy beverages
- [x] Test 6.2: Organic + fair-trade
- [x] Test 6.3: Low-sodium snacks

## Issues Found: 0 ✅

## Documentation Quality: ✅ APPROVED

All examples work as documented.
```

---

## 🎯 Final Sign-Off

Once all tests pass:

- ✅ API behavior matches documentation
- ✅ All examples are executable
- ✅ Response structures are accurate
- ✅ Parameter descriptions are correct
- ✅ Pagination works as described
- ✅ Filters behave as documented
- ✅ READY TO MERGE

