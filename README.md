<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Recipe App</title>
  <style>
    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background: #2b1d0e;
      color: #fff;
    }
    header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      background: #111;
      padding: 15px 30px;
    }
    header h1 {
      margin: 0;
      font-size: 24px;
    }
    .search-bar {
      display: flex;
    }
    .search-bar input {
      padding: 8px;
      font-size: 16px;
      border: none;
      border-radius: 4px 0 0 4px;
      outline: none;
    }
    .search-bar button {
      padding: 8px 15px;
      font-size: 16px;
      border: none;
      background: red;
      color: white;
      cursor: pointer;
      border-radius: 0 4px 4px 0;
      transition: background-color 0.2s ease;
    }
    .search-bar button:hover {
      background-color: darkred;
    }
    #viewHeader {
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 15px 30px;
    }
    #viewTitle {
      font-size: 20px;
      font-weight: bold;
      color: #f5a623;
    }
    #backButton {
      padding: 8px 15px;
      background: #f5a623;
      border: none;
      color: #111;
      font-weight: bold;
      border-radius: 5px;
      cursor: pointer;
      display: none;
      transition: background-color 0.2s ease;
    }
    #backButton:hover {
      background: #d48806;
    }
    .container {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      gap: 20px;
      padding: 0 30px 30px 30px;
    }
    .card {
      background: #fff;
      color: #000;
      border-radius: 10px;
      overflow: hidden;
      box-shadow: 0 4px 12px rgba(0,0,0,0.3);
      text-align: center;
      transition: transform 0.2s;
    }
    .card:hover {
      transform: scale(1.05);
    }
    .card img {
      width: 100%;
      height: 180px;
      object-fit: cover;
    }
    .card h3 {
      margin: 10px 0 5px;
    }
    .card p {
      margin: 5px 10px;
      font-size: 14px;
    }
    .card button {
      margin: 10px;
      padding: 10px 15px;
      background: red;
      border: none;
      color: white;
      font-weight: bold;
      cursor: pointer;
      border-radius: 5px;
      transition: background-color 0.2s ease;
    }
    .card button:hover {
      background: darkred;
    }
    @media (max-width: 1024px) {
      .container {
        grid-template-columns: repeat(2, 1fr);
      }
    }
    @media (max-width: 600px) {
      .container {
        grid-template-columns: 1fr;
      }
    }
    #recipeModal {
      display: none;
      position: fixed;
      z-index: 1000;
      left: 0; top: 0;
      width: 100%; height: 100%;
      overflow: auto;
      background-color: rgba(0,0,0,0.8);
      color: #000;
      font-size: 16px;
    }
    #recipeModalContent {
      background: #fff;
      margin: 60px auto;
      padding: 20px;
      border-radius: 10px;
      max-width: 600px;
      position: relative;
    }
    #modalCloseBtn {
      position: absolute;
      top: 10px; right: 10px;
      background: red;
      border: none;
      color: white;
      font-size: 18px;
      cursor: pointer;
      border-radius: 50%;
      width: 30px;
      height: 30px;
      line-height: 26px;
      text-align: center;
    }
    #recipeModalContent h2 {
      margin-top: 0;
    }
    #recipeModalContent img {
      max-width: 100%;
      border-radius: 10px;
      margin-bottom: 15px;
    }
    #recipeInstructions {
      white-space: pre-wrap;
      max-height: 300px;
      overflow-y: auto;
      margin-bottom: 15px;
    }
  </style>
</head>
<body>
  <header>
    <h1>RECIPE APP</h1>
    <div class="search-bar">
      <input type="text" id="searchInput" placeholder="Search recipes..." />
      <button id="searchButton" type="button">Search</button>
    </div>
  </header>

  <div id="viewHeader">
    <span id="viewTitle">Categories</span>
    <button id="backButton">⬅ Back</button>
  </div>

  <div class="container" id="recipeContainer">
  
  </div>

  <div id="recipeModal">
    <div id="recipeModalContent">
      <button id="modalCloseBtn" aria-label="Close modal">&times;</button>
      <h2 id="modalMealName"></h2>
      <img id="modalMealThumb" src="" alt="" />
      <div id="recipeInstructions"></div>
      <a id="modalYoutube" href="#" target="_blank" rel="noopener">Watch Video</a>
    </div>
  </div>

  <script>
    const container = document.getElementById("recipeContainer");
    const viewTitle = document.getElementById("viewTitle");
    const backButton = document.getElementById("backButton");
    const searchInput = document.getElementById("searchInput");
    const searchButton = document.getElementById("searchButton");
    const modal = document.getElementById("recipeModal");
    const modalCloseBtn = document.getElementById("modalCloseBtn");
    const modalMealName = document.getElementById("modalMealName");
    const modalMealThumb = document.getElementById("modalMealThumb");
    const recipeInstructions = document.getElementById("recipeInstructions");
    const modalYoutube = document.getElementById("modalYoutube");

    backButton.addEventListener("click", () => {
      loadCategories();
      backButton.style.display = "none";
    });

    async function loadCategories() {
      viewTitle.textContent = "Categories";
      backButton.style.display = "none";
      container.innerHTML = "<p>Loading categories...</p>";
      try {
        const response = await fetch("https://www.themealdb.com/api/json/v1/1/categories.php");
        const data = await response.json();
        if (!data.categories) {
          container.innerHTML = "<p>No categories found!</p>";
          return;
        }
        container.innerHTML = "";
        data.categories.forEach(category => {
          const card = document.createElement("div");
          card.className = "card";
          card.innerHTML = `
            <img src="${category.strCategoryThumb}" alt="${category.strCategory}" loading="lazy" />
            <h3>${category.strCategory}</h3>
            <p>${category.strCategoryDescription.substring(0, 100)}...</p>
            <button onclick="loadRecipesByCategory('${category.strCategory}')">View Recipes</button>
          `;
          container.appendChild(card);
        });
      } catch (error) {
        container.innerHTML = "<p>Error loading categories.</p>";
      }
    }

    async function loadRecipesByCategory(category) {
      searchInput.value = ""; // Clear search input
      viewTitle.textContent = `Category: ${category}`;
      backButton.style.display = "inline-block";
      container.innerHTML = `<p>Loading ${category} recipes...</p>`;
      try {
        const response = await fetch(`https://www.themealdb.com/api/json/v1/1/filter.php?c=${encodeURIComponent(category)}`);
        const data = await response.json();
        if (!data.meals) {
          container.innerHTML = "<p>No recipes found for this category.</p>";
          return;
        }
        container.innerHTML = "";
        data.meals.forEach(meal => {
          const card = document.createElement("div");
          card.className = "card";
          card.innerHTML = `
            <img src="${meal.strMealThumb}" alt="${meal.strMeal}" loading="lazy" />
            <h3>${meal.strMeal}</h3>
            <button onclick="viewRecipe('${meal.idMeal}')">View Recipe</button>
          `;
          container.appendChild(card);
        });
      } catch (error) {
        container.innerHTML = "<p>Error loading recipes.</p>";
      }
    }

    async function searchRecipe() {
      const query = searchInput.value.trim();
      if (query === "") {
        loadCategories();
        return;
      }
      viewTitle.textContent = `Search Results for "${query}"`;
      backButton.style.display = "inline-block";
      container.innerHTML = "<p>Searching recipes...</p>";
      try {
        const response = await fetch(`https://www.themealdb.com/api/json/v1/1/search.php?s=${encodeURIComponent(query)}`);
        const data = await response.json();
        if (!data.meals) {
          container.innerHTML = "<p>No recipes found!</p>";
          return;
        }
        container.innerHTML = "";
        data.meals.forEach(meal => {
          const card = document.createElement("div");
          card.className = "card";
          card.innerHTML = `
            <img src="${meal.strMealThumb}" alt="${meal.strMeal}" loading="lazy" />
            <h3>${meal.strMeal}</h3>
            <p><b>${meal.strArea}</b> Dish</p>
            <p>Category: <b>${meal.strCategory}</b></p>
            <button onclick="viewRecipe('${meal.idMeal}')">View Recipe</button>
          `;
          container.appendChild(card);
        });
      } catch (error) {
        container.innerHTML = "<p>Error searching recipes.</p>";
      }
    }

    async function viewRecipe(id) {
      try {
        const response = await fetch(`https://www.themealdb.com/api/json/v1/1/lookup.php?i=${id}`);
        const data = await response.json();
        if (!data.meals || data.meals.length === 0) {
          alert("Recipe details not found.");
          return;
        }
        const meal = data.meals[0];
        modalMealName.textContent = meal.strMeal;
        modalMealThumb.src = meal.strMealThumb;
        modalMealThumb.alt = meal.strMeal;
        recipeInstructions.textContent = meal.strInstructions;
        modalYoutube.href = meal.strYoutube || "#";
        modalYoutube.style.display = meal.strYoutube ? "inline-block" : "none";
        modal.style.display = "block";
      } catch (error) {
        alert("Failed to load recipe details.");
      }
    }

    modalCloseBtn.addEventListener("click", () => {
      modal.style.display = "none";
    });

    window.addEventListener("click", (e) => {
      if (e.target === modal) {
        modal.style.display = "none";
      }
    });

    searchButton.addEventListener("click", searchRecipe);

    searchInput.addEventListener("keydown", (e) => {
      if (e.key === "Enter") {
        searchRecipe();
      }
    });

    window.onload = loadCategories;
  </script>
</body>
</html>
