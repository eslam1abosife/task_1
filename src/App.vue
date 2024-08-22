<template>
  <div class="form-container">
    <!-- Main Category Dropdown -->
    <select
      v-model="selectedMainCategory"
      @change="fetchSubCategories"
      class="form-field"
    >
      <option value="" disabled>Select Main Category</option>
      <option
        v-for="category in mainCategories"
        :key="category.id"
        :value="category.id"
      >
        {{ category.name }}
      </option>
    </select>

    <!-- Sub Category Dropdown -->
    <select
      v-model="selectedSubCategory"
      @change="fetchProperties"
      class="form-field"
      v-if="subCategories.length"
    >
      <option value="" disabled>Select Sub Category</option>
      <option
        v-for="subCategory in subCategories"
        :key="subCategory.id"
        :value="subCategory.id"
      >
        {{ subCategory.name }}
      </option>
    </select>

    <!-- Properties Section -->
    <div
      v-for="(property, index) in properties"
      :key="property.id"
      class="property-field"
    >
      <label>{{ property.name }}</label>
      <select
        v-model="property.selectedOption"
        @change="handlePropertySelection(property, index)"
        class="form-field"
      >
        <option value="" disabled>Select {{ property.name }}</option>
        <option
          v-for="option in property.options"
          :key="option.id"
          :value="option.id"
        >
          {{ option.name }}
        </option>
        <option value="other">Other</option>
      </select>
      <input
        v-if="property.selectedOption === 'other'"
        v-model="property.otherValue"
        type="text"
        placeholder="Enter other value"
        class="form-field"
      />

      <!-- Child Properties -->
      <div v-if="property.childProperties.length">
        <div
          v-for="(childProperty, childIndex) in property.childProperties"
          :key="childProperty.id"
          class="child-property-field"
        >
          <label>{{ childProperty.name }}</label>
          <select
            v-model="childProperty.selectedChild"
            @change="handlePropertySelection(childProperty, childIndex)"
            class="form-field"
          >
            <option value="" disabled>Select {{ childProperty.name }}</option>
            <option
              v-for="option in childProperty.options"
              :key="option.id"
              :value="option.id"
            >
              {{ option.name }}
            </option>
            <option value="other">Other</option>
          </select>
          <input
            v-if="childProperty.selectedChild === 'other'"
            v-model="childProperty.otherValue"
            type="text"
            placeholder="Enter other value"
            class="form-field"
          />
        </div>
      </div>
    </div>

    <button @click="submitForm" class="submit-button">Submit</button>

    <table v-if="submittedData.length" class="results-table">
      <thead>
        <tr>
          <th>Key</th>
          <th>Value</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(data, index) in submittedData" :key="index">
          <td>{{ data.key }}</td>
          <td>{{ data.value }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      mainCategories: [],
      subCategories: [],
      selectedMainCategory: null,
      selectedSubCategory: null,
      properties: [],
      submittedData: [],
    };
  },
  mounted() {
    this.fetchMainCategories();
  },
  methods: {
    async fetchMainCategories() {
      try {
        const response = await axios.get(
          "https://staging.mazaady.com/api/v1/get_all_cats",
          {
            headers: { "private-key": "3%o8i}_;3D4bF]G5@22r2)Et1&mLJ4?$@+16" },
          }
        );
        this.mainCategories = response.data.data.categories.map((category) => ({
          id: category.id,
          name: category.name,
          children: category.children,
        }));
      } catch (error) {
        console.error("Error fetching main categories:", error);
      }
    },
    fetchSubCategories() {
      const selectedCategory = this.mainCategories.find(
        (category) => category.id === this.selectedMainCategory
      );

      if (selectedCategory && selectedCategory.children) {
        this.subCategories = selectedCategory.children.map((subCategory) => ({
          id: subCategory.id,
          name: subCategory.name,
        }));
      } else {
        this.subCategories = [];
      }
    },
    async fetchProperties() {
      if (!this.selectedSubCategory) return;

      try {
        const response = await axios.get(
          `https://staging.mazaady.com/api/v1/properties?cat=${this.selectedSubCategory}`,
          {
            headers: { "private-key": "3%o8i}_;3D4bF]G5@22r2)Et1&mLJ4?$@+16" },
          }
        );
        this.properties = response.data.data.map((property) => ({
          id: property.id,
          name: property.name,
          options: [...property.options, { id: "other", name: "Other" }],
          childProperties: [],
          selectedOption: null,
          otherValue: null,
        }));
      } catch (error) {
        console.error("Error fetching properties:", error);
      }
    },
    async handlePropertySelection(property, index) {
      if (property.selectedOption === "other") {
        property.otherValue = "";
      }

      // Fetch child properties if selected option has children
      if (property.selectedOption && property.selectedOption !== "other") {
        try {
          const response = await axios.get(
            `https://staging.mazaady.com/api/v1/get-options-child/${property.selectedOption}`,
            {
              headers: {
                "private-key": "3%o8i}_;3D4bF]G5@22r2)Et1&mLJ4?$@+16",
              },
            }
          );

          // بدلاً من استخدام this.$set، يمكنك تحديث الكائن مباشرةً
          const updatedProperty = {
            ...property,
            childProperties: response.data.data.map((child) => ({
              id: child.id,
              name: child.name,
              options: [...child.options, { id: "other", name: "Other" }],
              selectedChild: null,
              otherValue: null,
            })),
          };

          // قم بتحديث الخصائص في this.properties
          this.properties[index] = updatedProperty;
        } catch (error) {
          console.error("Error fetching child properties:", error);
        }
      }
    },
    submitForm() {
      this.submittedData = [
        {
          key: "Main Category",
          value: this.mainCategories.find(
            (category) => category.id === this.selectedMainCategory
          )?.name,
        },
        {
          key: "Sub Category",
          value: this.subCategories.find(
            (subCategory) => subCategory.id === this.selectedSubCategory
          )?.name,
        },
        ...this.properties.flatMap((property) => [
          {
            key: property.name,
            value:
              property.selectedOption === "other"
                ? property.otherValue
                : property.options.find(
                    (option) => option.id === property.selectedOption
                  )?.name,
          },
          ...property.childProperties.map((child) => ({
            key: child.name,
            value:
              child.selectedChild === "other"
                ? child.otherValue
                : child.options.find(
                    (option) => option.id === child.selectedChild
                  )?.name,
          })),
        ]),
      ];
    },
  },
};
</script>

<style>
.form-container {
  max-width: 600px;
  margin: 146px auto;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 10px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.form-field {
  width: 100%;
  margin-bottom: 15px;
  padding: 10px;
  border-radius: 5px;
  border: 1px solid #ccc;
  box-shadow: inset 0 1px 3px rgba(0, 0, 0, 0.1);
}

.submit-button {
  display: inline-block;
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.submit-button:hover {
  background-color: #0056b3;
}

.results-table {
  width: 100%;
  margin-top: 30px;
  border-collapse: collapse;
}

.results-table th,
.results-table td {
  padding: 10px;
  border: 1px solid #ddd;
  text-align: left;
}

.results-table th {
  background-color: #f1f1f1;
}
</style>
