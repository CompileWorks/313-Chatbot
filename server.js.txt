import 'dotenv/config';
import express from 'express';
import cors from 'cors';
import OpenAI from 'openai';

const app = express();
app.use(cors());
app.use(express.json());
app.use(express.static('public')); // serves frontend from /public

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

// Health check
app.get('/api/health', (_req, res) => {
  res.json({ ok: true, bot: '313' });
});

// Chat endpoint
app.post('/api/chat', async (req, res) => {
  try {
    const { messages } = req.body;
    if (!Array.isArray(messages) || messages.length === 0) {
      return res.status(400).json({ error: 'messages[] required' });
    }

    const completion = await openai.chat.completions.create({
      model: 'gpt-4.1-mini',
      messages: [
        {
          role: 'system',
          content:
            'You are 313, a concise, friendly AI assistant. Keep replies helpful and safe.'
        },
        ...messages
      ],
      temperature: 0.7,
      max_tokens: 512
    });

    const reply =
      completion.choices?.[0]?.message?.content?.trim() || '(No response)';
    res.json({ reply });
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: err?.message || 'Server error' });
  }
});

const port = process.env.PORT || 3000;
app.listen(port, () => {
  console.log(`313 server running on http://localhost:${port}`);
});
