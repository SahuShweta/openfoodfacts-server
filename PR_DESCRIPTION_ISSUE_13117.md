# Pull Request: Improve Search API Documentation for Filters & Pagination

**Related Issue:** #13117

## 🎯 Purpose

This PR significantly improves the Search API documentation in the Open Food Facts server repository by providing comprehensive, structured guidance on filters and pagination parameters.

## 📋 Changes

### New Documentation
- **[docs/api/search-parameters.md](docs/api/search-parameters.md)** — A comprehensive guide covering:
  - Pagination (`page`, `page_size`) with defaults and edge cases
  - All 15 tag-based filters with language suffixes
  - Logical operators (AND, OR, NOT) with clear examples
  - Nutrient filtering syntax and comparison operators
  - Sorting options (11 documented sort criteria)
  - Response field limiting strategies
  - 4 complete, real-world search examples
  - Rate limiting and environment information

### Enhanced Documentation
- **docs/api/ref-cheatsheet.md** — Addition of quick examples and reference link to comprehensive search guide

## 📊 What Was Previously Unclear

| Problem | Solution |
|---------|----------|
| No single source of truth for search filters | Complete filter catalog with examples |
| Pagination defaults and edge cases undocumented | Documented defaults (page=1, page_size=24) and edge case handling |
| Logical operators (AND/OR/NOT) not documented | Clear section with syntax and examples |
| Nutrient filter syntax unclear | Full syntax documentation: `{nutrient}_{base}{operator}{value}` |
| Only 2 sorting options mentioned | All 11 sorting options documented |
| `page_count` response field "counter intuitive" | All response fields explained clearly |
| Scattered filter information across multiple docs | Centralized comprehensive reference |

## ✅ Documentation Verification

All parameters documented are verified against the OpenAPI specification (`docs/api/ref/api.yaml`):

### Documented Parameters
- **Pagination:** 2 parameters (`page`, `page_size`)
- **Tag Filters:** 15 filters with language suffix support
- **Nutrient Filters:** Syntax, bases, and operators
- **Sorting:** 11 `sort_by` enum values
- **Response:** Fields parameter for limiting response size

### Example Endpoints (Tested)
```bash
# Healthy beverages (tag + nutrient filters + sorting)
curl "https://world.openfoodfacts.org/api/v2/search?categories_tags_en=beverages&nutrition_grades_tags=a|b&sugars_100g<5&sort_by=nutriscore_score"

# Organic + fair-trade products (AND operator)
curl "https://world.openfoodfacts.org/api/v2/search?labels_tags=en:organic,en:fair-trade"

# Low-sodium snacks (nutrient + pagination)
curl "https://world.openfoodfacts.org/api/v2/search?categories_tags_en=snacks&salt_100g<0.3&page=1&page_size=25"

# Recent products (sorting + pagination)
curl "https://world.openfoodfacts.org/api/v2/search?sort_by=last_modified_t&page_size=100"
```

## 🔄 Backwards Compatibility

✅ **Fully backwards compatible** — This is purely documentation improvement.

- No API behavior changes
- No endpoint modifications
- No parameter removals
- All existing calls continue working

## 📖 Documentation Quality

The new documentation follows repository conventions:
- ✅ Markdown formatting consistent with existing docs
- ✅ Relative links for cross-references
- ✅ Tables, code blocks, and visual hierarchy
- ✅ Quick examples before detailed reference
- ✅ Real-world use cases
- ✅ Edge case documentation
- ✅ Performance notes (e.g., use `fields` parameter to reduce bandwidth)

## 🧪 How to Review

1. **Rendering:** Check markdown renders correctly on docs site
2. **Links:** Verify all cross-references work
3. **Examples:** Test 3-4 example URLs against staging API
4. **Accuracy:** Confirm parameter descriptions match actual API behavior
5. **Completeness:** Verify all 15 tag filters are listed
6. **Integration:** Check ref-cheatsheet.md update flows naturally

## 📚 Files in This PR

```
docs/api/search-parameters.md          [NEW] ~850 lines
docs/api/ref-cheatsheet.md             [ENHANCED] addition to Search for Products section
```

## 🎓 Impact

### For API Users
- Clear, centralized documentation reduces support questions
- Examples enable faster API integration
- Edge case documentation prevents common mistakes
- Performance notes help optimize API usage

### For API Maintainers
- Single source of truth for search functionality
- Easier to update when new filters are added
- Reduces documentation debt and scattered information

### For Contributors
- Clear examples for how to document new filters
- Searchable parameter reference for feature development

## 🚀 Next Steps

1. Review this PR and validate examples
2. Merge to main branch
3. Update docs website to build and deploy new documentation
4. Close related issue #13117

## ✨ Additional Notes

- All example URLs use valid production endpoints (world.openfoodfacts.org)
- Response structures shown match actual API responses
- No changes to documentation in other sections
- Focused solely on Search API (v2) documentation
- Full-text search alternatives clearly referenced

---

**Reviewer Checklist:**
- [ ] Markdown renders correctly
- [ ] All links are valid and working
- [ ] At least 2 example URLs tested and confirmed working
- [ ] No merge conflicts with current main
- [ ] Parameter documentation matches API spec
- [ ] Examples are realistic and useful
- [ ] Ready to merge

