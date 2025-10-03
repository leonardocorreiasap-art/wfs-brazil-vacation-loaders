# 🏖️ WFS Brazil Vacation Loaders

Ferramentas para geração de arquivos de carga inicial de férias no **Workforce Software (WFS)** conforme legislação brasileira (CLT).

---

## 🎯 Ferramentas Disponíveis

### 1. **LD57 - Vacation Initialization Generator**
[**🔗 Abrir Ferramenta**](./ld57-generator.html)

Gera arquivo de inicialização de períodos aquisitivos de férias.

**Funcionalidades:**
- ✅ Cálculo automático de período aquisitivo vigente na data de Go-Live
- ✅ 12 colunas conforme layout WFS oficial
- ✅ 2 linhas de header padrão
- ✅ Exclusão automática de estagiários (PP7)
- ✅ Remoção de registros duplicados
- ✅ Employee ID formatado com 4 dígitos (0001, 0047)
- ✅ Formato de data: dd/MM/yyyy

**Arquivo gerado:** `EXT_BRA_VACATION_INITIALIZATION_DATES_[data].csv`

---

### 2. **Bank Balance Import Generator**
[**🔗 Abrir Ferramenta**](./bank-balance-generator.html)

Gera arquivo de saldos iniciais para importação de bancos de férias.

**Funcionalidades:**
- ✅ Cálculo automático de saldos (Accrual, Entitlement, Carryover, 13º)
- ✅ Layout padrão WFS: EMPLOYEE_ID, BANK, BALANCE, AS_OF_DATE
- ✅ Opção para incluir banco CARRYOVER (férias acumuladas)
- ✅ Opção para incluir banco 13º SALÁRIO (adiantamentos)
- ✅ Formato de data: yyyy-MM-dd (ISO 8601)
- ✅ Preview em tempo real

**Arquivo gerado:** `BANK_BALANCE_IMPORT_[data].csv`

---

## 🚀 Como Usar

### **Requisitos**
- Navegador web moderno (Chrome, Firefox, Safari, Edge)
- Arquivo `emp_import.csv` com dados dos funcionários

### **Formato do Arquivo de Entrada**

Seu arquivo `emp_import.csv` deve ter no mínimo estas colunas:

```csv
EXTERNAL_HR_ID,LATEST_ASSIGNMENT_BEGIN_DATE,POLICY_PROFILE
1,2023-02-10,PP1
2,2023-02-10,PP2
47,2023-01-20,PP3
```

| Coluna | Descrição | Formato |
|--------|-----------|---------|
| `EXTERNAL_HR_ID` | ID do funcionário | Numérico |
| `LATEST_ASSIGNMENT_BEGIN_DATE` | Data de admissão | YYYY-MM-DD ou DD/MM/YYYY |
| `POLICY_PROFILE` | Perfil de política | PP1, PP2, etc. |

---

## 📋 Passo a Passo - Implementação de Go-Live

### **1️⃣ Gerar LD57 (Períodos Aquisitivos)**

1. Abra **[ld57-generator.html](./ld57-generator.html)** no navegador
2. Configure a **data de Go-Live** (padrão: 01/08/2025)
3. Faça **upload** do arquivo `emp_import.csv`
4. Aguarde o processamento automático
5. Revise as **estatísticas** e preview dos dados
6. Click em **"Baixar CSV"**
7. Importe no WFS via **LD Upload Utility**

**Path WFS:** `Administration → Data Management → LD Upload Utility → LD57`

---

### **2️⃣ Gerar Bank Balance (Saldos)**

1. Abra **[bank-balance-generator.html](./bank-balance-generator.html)** no navegador
2. Configure a **mesma data de Go-Live**
3. Faça **upload** do mesmo `emp_import.csv`
4. Marque opções conforme necessário:
   - ☐ **Incluir CARRYOVER** (somente se houver férias vencidas a migrar)
   - ☐ **Incluir 13º SALÁRIO** (somente se WFS gerenciará adiantamentos)
5. Click em **"Baixar CSV"**
6. Importe no WFS via **Bank Balance Import**

**Path WFS:** `Administration → Jobs and Data Management → Imports → Bank Balance Import`

---

### **3️⃣ Calcular Timesheets**

Após importar ambos os arquivos:

1. Acesse **Timesheet → Calculate**
2. Selecione período: **data de Go-Live até data atual**
3. Execute o cálculo
4. Valide os saldos nos relatórios de bancos

---

## 📊 Bancos de Férias Configurados

| Código do Banco | Descrição | Cálculo |
|---|---|---|
| `EXT_BRA_VACATION_ACCRUAL_BANK` | Aquisição de Férias | 2,5 dias/mês trabalhado |
| `EXT_BRA_VACATION_ENTITLEMENT_BANK` | Direito a Férias | 30 dias após 12 meses |
| `EXT_BRA_VACATION_CARRYOVER_BANK` | Férias Acumuladas | Períodos anteriores (opcional) |
| `EXT_BRA_VACATION_THIRTEENTH_MONTH_ADVANCE_BANK` | 13º Salário | 1.0 adiantamento/ano (opcional) |

---

## 🎯 Lógica de Cálculo - CLT

### **Banco Accrual (Aquisição)**
```
Cálculo: 2,5 dias por mês trabalhado no período aquisitivo atual
Exemplo: Admitido em 01/02/2025, Go-Live 01/08/2025
  → 6 meses trabalhados × 2,5 = 15 dias
```

### **Banco Entitlement (Direito)**
```
Cálculo: 30 dias quando completa 12 meses de período aquisitivo
Transferência: Automática do banco Accrual após 12 meses
```

### **Banco Carryover (Acumulado)** ⚠️ Opcional
```
Propósito: Férias de períodos anteriores não utilizadas
⚠️ IMPORTANTE: Marque SOMENTE se houver certeza de que precisa 
   carregar férias vencidas. Na maioria dos Go-Lives, começar vazio.
```

### **Banco 13º Salário** ⚠️ Opcional
```
Valor inicial: 1.0 (um adiantamento disponível por ano)
⚠️ IMPORTANTE: Marque se o WFS gerenciará adiantamentos do 13º salário
```

---

## ⚙️ Configurações Importantes

### **Data de Go-Live**
- Padrão: **01/08/2025** (editável em ambas ferramentas)
- Deve ser a **mesma data** em LD57 e Bank Balance
- Representa o início da operação no WFS

### **Exclusões Automáticas**
- **Estagiários (PP7):** Removidos automaticamente
- **Registros duplicados:** Removidos automaticamente
- **Datas inválidas:** Excluídas do processamento

### **Formatação de IDs**
- Employee IDs são formatados com 4 dígitos: `0001`, `0047`, `0123`
- Garante compatibilidade com importadores WFS

---

## ⚠️ Ordem de Execução (CRÍTICO!)

> **🎯 IMPORTANTE:** Sempre siga esta ordem:

```
1. Importar LD57 (inicializa períodos aquisitivos)
       ↓
2. Importar Bank Balance (carrega saldos)
       ↓
3. Calcular Timesheets (processa regras)
```

Inverter esta ordem pode causar erros de validação no WFS!

---

## 🐛 Troubleshooting

### **Erro: "Invalid date format"**
✅ Verifique se as datas no `emp_import.csv` estão em `YYYY-MM-DD` ou `DD/MM/YYYY`

### **Erro: "Employee ID not found"**
✅ Certifique-se de que todos os IDs são numéricos e únicos

### **Saldos não aparecem após import**
✅ Soluções:
1. Confirme que LD57 foi importado **ANTES** do Bank Balance
2. Execute o **cálculo de timesheet** para o período
3. Verifique **logs de importação** no WFS

### **Estagiários aparecendo nos arquivos**
✅ Verifique se o `POLICY_PROFILE` está como `PP7` no arquivo de entrada

### **Preview mostra dados corretos, mas CSV baixado está vazio**
✅ Desabilite bloqueadores de popup e verifique a pasta de Downloads

---

## 💻 Modo de Uso Local

Ambas as ferramentas funcionam **100% offline** no navegador:

1. Baixe os arquivos `.html`
2. Abra diretamente no navegador (duplo-clique)
3. Nenhuma instalação necessária
4. Nenhum dado é enviado para servidores externos

**Bibliotecas utilizadas (via CDN):**
- React 18
- Tailwind CSS
- PapaParse (processamento CSV)
- Lucide Icons

---

## 📄 Licença

MIT License - Livre para uso e modificação

---

## 👤 Autor

**Leonardo Correia**  
🔗 GitHub: [@leonardocorreiasap-art](https://github.com/leonardocorreiasap-art)

---

## 🎓 Sobre

Ferramentas desenvolvidas por **Consultor Sênior WFS** com 20 anos de experiência em Time & Attendance, focadas em:

- ✅ Conformidade com legislação trabalhista brasileira (CLT)
- ✅ Boas práticas de implementação WFS
- ✅ Automação de processos de Go-Live
- ✅ Redução de erros manuais em cargas iniciais

---

**Última atualização:** Outubro 2025  
**Versão:** 2.0.0