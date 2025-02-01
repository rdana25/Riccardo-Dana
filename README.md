import express from 'express';
import axios from 'axios';
import dotenv from 'dotenv';

dotenv.config();

const app = express();
const port = process.env.PORT || 3000;

app.use(express.json());

// Function to query ChatGPT
async function queryChatGPT(prompt) {
    const response = await axios.post('https://api.openai.com/v1/chat/completions', {
        model: 'gpt-4',
        messages: [{ role: 'user', content: prompt }]
    }, {
        headers: { 'Authorization': `Bearer ${process.env.OPENAI_API_KEY}` }
    });
    return response.data.choices[0].message.content;
}

// Function to query Claude (Anthropic API)
async function queryClaude(prompt) {
    const response = await axios.post('https://api.anthropic.com/v1/complete', {
        model: 'claude-2',
        prompt,
        max_tokens: 256
    }, {
        headers: { 'Authorization': `Bearer ${process.env.ANTHROPIC_API_KEY}`, 'Content-Type': 'application/json' }
    });
    return response.data.completion;
}

// Function to query DeepSeek AI
async function queryDeepSeek(prompt) {
    const response = await axios.post('https://api.deepseek.com/v1/generate', {
        model: 'deepseek-chat',
        prompt,
        max_tokens: 256
    }, {
        headers: { 'Authorization': `Bearer ${process.env.DEEPSEEK_API_KEY}` }
    });
    return response.data.choices[0].text;
}

// Route to interact with Chappie AI
app.post('/chappie', async (req, res) => {
    const { prompt, model } = req.body;

    try {
        let response;
        switch (model) {
            case 'chatgpt':
                response = await queryChatGPT(prompt);
                break;
            case 'claude':
                response = await queryClaude(prompt);
                break;
            case 'deepseek':
                response = await queryDeepSeek(prompt);
                break;
            default:
                return res.status(400).json({ error: 'Invalid model specified' });
        }

        res.json({ response });
    } catch (error) {
        res.status(500).json({ error: 'API request failed', details: error.message });
    }
});

app.listen(port, () => {
    console.log(`Chappie AI server running on port ${port}`);
});
