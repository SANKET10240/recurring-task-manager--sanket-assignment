# recurring-task-manager--sanket-assignment
A lightweight web application to manage recurring tasks with email reminders.
# Recurring Task Reminder Manager

## Project Description
The Recurring Task Reminder Manager is a lightweight web application designed to help users efficiently track and manage recurring tasks. It sends email notifications based on customized schedules, making it ideal for freelancers, small business owners, and individuals who want to stay on top of routine responsibilities.

## Features
- Add, edit, and delete recurring tasks.
- Set customizable intervals (daily, weekly, monthly, yearly).
- Receive email reminders for upcoming tasks.
- View and manage tasks in an intuitive dashboard.

## Screenshots
### Dashboard
![Dashboard](screenshots/dashboard.png)
### Task Creation
![Task Creation](screenshots/task_creation.png)
### Email Notification
![Email Notification](screenshots/email_notification.png)

React Code (App.js)
import React, { useState, useEffect } from "react";

function App() {
  const [tasks, setTasks] = useState([]);
  const [form, setForm] = useState({ name: "", description: "", frequency: "", email: "" });

  useEffect(() => {
    // Fetch existing tasks from the backend (optional)
    fetch("http://localhost:3000/tasks")
      .then((res) => res.json())
      .then((data) => setTasks(data))
      .catch((err) => console.error("Error fetching tasks:", err));
  }, []);

  const handleInputChange = (e) => {
    const { name, value } = e.target;
    setForm({ ...form, [name]: value });
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      const response = await fetch("http://localhost:3000/tasks", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(form),
      });
      const newTask = await response.json();
      setTasks([...tasks, newTask]);
      setForm({ name: "", description: "", frequency: "", email: "" });
    } catch (error) {
      console.error("Error adding task:", error);
    }
  };

  return (
    <div>
      <h1>Recurring Task Reminder Manager</h1>
      <form onSubmit={handleSubmit}>
        <input name="name" placeholder="Task Name" value={form.name} onChange={handleInputChange} required />
        <input name="description" placeholder="Description" value={form.description} onChange={handleInputChange} required />
        <select name="frequency" value={form.frequency} onChange={handleInputChange} required>
          <option value="">Select Frequency</option>
          <option value="daily">Daily</option>
          <option value="weekly">Weekly</option>
          <option value="monthly">Monthly</option>
          <option value="yearly">Yearly</option>
        </select>
        <input name="email" placeholder="Your Email" value={form.email} onChange={handleInputChange} required />
        <button type="submit">Add Task</button>
      </form>
      <ul>
        {tasks.map((task, index) => (
          <li key={index}>{`${task.name} - ${task.frequency}`}</li>
        ))}
      </ul>
    </div>
  );
}

export default App;


Liscense (MIT Liscense)

---

### **Code Syntax Corrections**

#### Backend (`server.js`)
Corrected the backend Node.js code for syntax issues and logical errors:

```javascript
const express = require("express");
const mongoose = require("mongoose");
const nodemailer = require("nodemailer");
const dotenv = require("dotenv");

dotenv.config();
const app = express();
app.use(express.json());

// MongoDB connection
mongoose.connect("mongodb://127.0.0.1:27017/taskManager", {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});

// Task Schema
const taskSchema = new mongoose.Schema({
  name: { type: String, required: true },
  description: { type: String, required: true },
  frequency: { type: String, enum: ["daily", "weekly", "monthly", "yearly"], required: true },
  email: { type: String, required: true },
  nextReminder: { type: Date, required: true },
});

const Task = mongoose.model("Task", taskSchema);

// Nodemailer configuration
const transporter = nodemailer.createTransport({
  service: process.env.EMAIL_SERVICE,
  auth: {
    user: process.env.EMAIL_USER,
    pass: process.env.EMAIL_PASS,
  },
});

// API to create a task
app.post("/tasks", async (req, res) => {
  try {
    const { name, description, frequency, email } = req.body;
    const nextReminder = new Date(); // For simplicity, reminder starts immediately
    const task = new Task({ name, description, frequency, email, nextReminder });
    await task.save();
    res.status(201).json(task);
  } catch (error) {
    res.status(500).json({ message: "Error creating task", error: error.message });
  }
});

// Send email reminders (run as a cron job or manually trigger)
app.get("/send-reminders", async (req, res) => {
  try {
    const tasks = await Task.find();
    tasks.forEach(async (task) => {
      const mailOptions = {
        from: process.env.EMAIL_USER,
        to: task.email,
        subject: `Reminder: ${task.name}`,
        text: `Don't forget to: ${task.description}`,
      };
      await transporter.sendMail(mailOptions);
    });
    res.send("Reminders sent.");
  } catch (error) {
    res.status(500).json({ message: "Error sending reminders", error: error.message });
  }
});

// Start the server
app.listen(3000, () => console.log("Server running on http://localhost:3000"));




