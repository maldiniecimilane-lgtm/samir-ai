# S AI 🇲🇦

Welcome to S AI – Moroccan AI Assistant.

An intelligent assistant designed to help users with AI-powered conversations.

## Features
- AI chat assistant
- Simple interface
- Moroccan identity

## Project
S AI – 2026 

import express from "express";
import { GoogleGenAI } from "@google/genai";

const app = express();
const PORT = process.env.PORT || 3000;

app.use(express.json({ limit: "20mb" }));

const apiKey = process.env.GEMINI_API_KEY;

app.get("/", (req, res) => {
  res.json({
    status: "success",
    project: "S AI Morocco",
    message: "Welcome to S AI 🇲🇦"
  });
});

app.post("/api/chat", async (req, res) => {
  try {
    if (!apiKey) {
      return res.status(500).json({
        error: "Gemini API key is missing"
      });
    }

    const { messages } = req.body;

    if (!Array.isArray(messages) || messages.length === 0) {
      return res.status(400).json({
        error: "Messages are required"
      });
    }

    const ai = new GoogleGenAI({ apiKey });

    const contents = messages.map((m) => {
      const parts = [{ text: String(m.content || "") }];

      if (Array.isArray(m.images)) {
        for (const image of m.images) {
          parts.push({
            inlineData: {
              mimeType: "image/jpeg",
              data: image
            }
          });
        }
      }

      return {
        role: m.role === "assistant" ? "model" : "user",
        parts
      };
    });

    const response = await ai.models.generateContent({
      model: "gemini-flash-latest",
      contents,
      config: {
        systemInstruction:
          "You are S AI, a helpful multilingual AI assistant from Morocco. Reply naturally in the user's language."
      }
    });

    res.json({
      text: response.text || "لم أتمكن من إنشاء إجابة."
    });

  } catch (error) {
    console.error("AI Error:", error.message);

    res.status(500).json({
      error: "AI service error"
    });
  }
});

app.listen(PORT, () => {
  console.log(`S AI running on port ${PORT}`);
});
