# Operação reCAPTCHA — Repositório de Correções
## Sino.Sistema.Cordeirópolis.Web — Formulário "Fale Conosco"

**Data:** 2026
**Status:** ✅ RESOLVIDO

---

### Resumo

O formulário "Fale Conosco" de Cordeirópolis apresentava os mesmos problemas de validação identificados em Novo Horizonte. A correção seguiu o mesmo padrão, porém com uma diferença importante: além do ajuste no **back-end**, foi necessário corrigir também o **front-end**, adicionando os atributos corretos nos campos de entrada do formulário.

---

### Problemas Identificados e Resoluções

#### 🔹 Back-end — `FaleConosco/Index.cshtml.cs`

**1. Bloqueio falso no `ModelState.IsValid` por `[BindProperty]`**

- **Causa Raiz:** A propriedade `ErrorMessage` possuía a anotação `[BindProperty]`, fazendo com que o .NET (com *Nullable Reference Types* habilitado) exigisse o preenchimento do campo no envio do formulário, bloqueando a validação indevidamente.
- **Correção:** A anotação `[BindProperty]` foi comentada e o valor inicial foi definido como `string.Empty`:

```csharp
// [BindProperty] -> Anotação comentada
public string ErrorMessage { get; set; } = string.Empty;
```

**2. Ausência de validação do token reCAPTCHA**

- **Causa Raiz:** Não havia verificação do token `g-recaptcha-response` antes do processamento do `ModelState`, permitindo submissão sem interação com o reCAPTCHA.
- **Correção:** Adicionada verificação explícita do token, forçando erro de validação caso o campo esteja vazio:

```csharp
// Verifica se o usuário clicou no reCAPTCHA (o token foi gerado)
var recaptchaResponse = Request.Form["g-recaptcha-response"].ToString();
if (string.IsNullOrEmpty(recaptchaResponse))
{
    // Força um erro no ModelState
    ModelState.AddModelError(string.Empty, "Por favor, marque a caixa 'Não sou um robô' para continuar.");
}

if (!ModelState.IsValid)
{
    return Page();
}
```

**3. Limpeza do formulário após envio bem-sucedido**

- Após o envio com sucesso, o formulário é reiniciado corretamente: a flag de sucesso é ativada, o `ModelState` é limpo e uma nova instância vazia do modelo é criada:

```csharp
SuccessMessage = true;
ModelState.Clear();
// Instancia um novo formulário vazio
FaleConosco = new Faleconosco();
return Page();
```

---

#### 🔹 Front-end — `FaleConosco/Index.cshtml`

- **Causa Raiz:** Os campos de entrada do formulário não possuíam os atributos `required` e `autocomplete`, o que prejudicava tanto a validação nativa do navegador quanto a experiência do usuário (preenchimento automático).
- **Correção:** Adicionados os atributos `required` e `autocomplete` apropriados em cada campo:

```html
<input type="text"
       class="form-control"
       asp-for="FaleConosco.Nome"
       required
       autocomplete="name" />

<input type="email"
       class="form-control"
       asp-for="FaleConosco.Email"
       required
       autocomplete="email" />

<input type="tel"
       class="form-control"
       asp-for="FaleConosco.Telefone"
       autocomplete="tel" />

<textarea class="form-control"
          asp-for="FaleConosco.Mensagem"
          rows="4"
          required></textarea>
```

---

### Matriz de Arquivos Modificados

| Arquivo | Tipo | Modificação |
| :--- | :--- | :--- |
| `FaleConosco/Index.cshtml.cs` | Código C# | Comentado `[BindProperty]` em `ErrorMessage`; adicionada verificação do token reCAPTCHA; implementada limpeza do formulário pós-envio |
| `FaleConosco/Index.cshtml` | Front-end HTML | Adicionados atributos `required` e `autocomplete` nos campos do formulário |

---

### Diferencial em Relação ao Padrão de Novo Horizonte

| Ponto | Novo Horizonte | Cordeirópolis — Fale Conosco |
| :--- | :--- | :--- |
| Correção no back-end | ✅ Sim | ✅ Sim |
| Correção no front-end | ❌ Não necessário | ✅ Sim (atributos `required` e `autocomplete`) |
| Validação reCAPTCHA via `ModelState` | ✅ Sim | ✅ Sim |
| Limpeza do formulário pós-envio | — | ✅ Sim (`ModelState.Clear` + nova instância) |

---

### Padrão de Implementação Recomendado

Este caso reforça que, além da correção padrão no back-end, **o front-end deve ser revisado** em cada formulário migrado ou adaptado. A ausência de `required` nos campos pode mascarar erros de validação e comprometer a experiência do cidadão. Recomenda-se incluir esta verificação como checklist padrão no processo de homologação de formulários da rede Sino.