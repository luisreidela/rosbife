const express = require('express');
const sqlite3 = require('sqlite3').verbose();
const bodyParser = require('body-parser');

const app = express();
const PORT = 3000;

// Middleware
app.use(bodyParser.json());

// Banco de dados SQLite
const db = new sqlite3.Database('./controle_portaria.db', (err) => {
    if (err) console.error('Erro ao conectar ao banco de dados:', err.message);
    console.log('Conectado ao banco SQLite.');
});

db.run(`
    CREATE TABLE IF NOT EXISTS visitantes (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        nome TEXT,
        cpf TEXT,
        placa TEXT,
        hora_entrada TEXT
    )
`);

// Rota para registrar visitante
app.post('/registrar', (req, res) => {
    const { nome, cpf, placa } = req.body;

    // Validação básica de CPF e placa
    if (cpf.length !== 11 || placa.length < 7) {
        return res.json({ mensagem: 'CPF ou placa inválidos.' });
    }

    const horaEntrada = new Date().toISOString();
    db.run(
        `INSERT INTO visitantes (nome, cpf, placa, hora_entrada) VALUES (?, ?, ?, ?)`,
        [nome, cpf, placa, horaEntrada],
        function (err) {
            if (err) {
                return res.status(500).json({ mensagem: 'Erro ao registrar visitante.' });
            }
            res.json({ mensagem: 'Visitante registrado com sucesso!' });
        }
    );
});

// Iniciar servidor
app.listen(PORT, () => {
    console.log(`Servidor rodando em http://localhost:${PORT}`);
});
