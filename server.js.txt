require("dotenv").config();
const express = require("express");
const jwt = require("jsonwebtoken");
const cors = require("cors");
const morgan = require("morgan");

const app = express();

// Configuration
const PORT = process.env.PORT || 5000;
const SECRET_KEY = process.env.SECRET_KEY || "your_secret_key";

// Middleware
app.use(cors());
app.use(express.json());
app.use(morgan("dev"));

// Mock Database
const users = [
  { username: "admin", password: "password" },
  { username: "developer", password: "devpass" },
];

// Utility Functions
const generateToken = (username) => {
  return jwt.sign({ username }, SECRET_KEY, { expiresIn: "1h" });
};

const authenticateUser = (username, password) => {
  return users.find((user) => user.username === username && user.password === password);
};

const verifyToken = (req, res, next) => {
  const token = req.headers["authorization"];
  if (!token) {
    return res.status(403).json({ error: "No token provided" });
  }
  jwt.verify(token, SECRET_KEY, (err, decoded) => {
    if (err) {
      return res.status(401).json({ error: "Invalid or expired token" });
    }
    req.user = decoded;
    next();
  });
};

// Routes

// Login Route
app.post("/auth/login", (req, res) => {
  const { username, password } = req.body;

  if (!username || !password) {
    return res.status(400).json({ error: "Username and password are required" });
  }

  const user = authenticateUser(username, password);

  if (!user) {
    return res.status(401).json({ error: "Invalid username or password" });
  }

  const token = generateToken(username);
  res.json({ message: "Login successful", token });
});

// Analyze Code Route
app.post("/analyze", verifyToken, (req, res) => {
  const { code, language } = req.body;

  if (!code || !language) {
    return res.status(400).json({ error: "Code and language are required" });
  }

  const analysis = performCodeAnalysis(code, language);

  if (!analysis) {
    return res.status(400).json({ error: `Unsupported language: ${language}` });
  }

  res.json({ language, analysis });
});

// Health Check Route
app.get("/health", (req, res) => {
  res.json({ status: "Server is running", timestamp: new Date().toISOString() });
});

// Helper Function for Code Analysis
const performCodeAnalysis = (code, language) => {
  const recommendations = {
    javascript: `JavaScript Analysis:
- Use 'let' or 'const' for variable declarations.
- Avoid using 'var'.
- Ensure proper use of async/await for asynchronous code.`,
    python: `Python Analysis:
- Check for proper indentation.
- Use comprehensions for loops when possible.
- Avoid using global variables unless necessary.`,
    java: `Java Analysis:
- Handle exceptions properly using try-catch blocks.
- Ensure use of private fields with getters and setters.
- Avoid redundant code in methods.`,
    cpp: `C++ Analysis:
- Use smart pointers to manage memory.
- Avoid memory leaks by releasing unused memory.
- Ensure all variables are initialized before use.`,
  };

  return recommendations[language.toLowerCase()] || null;
};

// Start the Server
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
