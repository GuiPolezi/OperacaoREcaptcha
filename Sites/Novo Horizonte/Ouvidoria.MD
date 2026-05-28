# Relatório Técnico — Validação de reCAPTCHA nos Formulários
## Projeto: Sino.Sistema.NovoHorizonte.Web

**Data:** 28 de Maio de 2026
**Status:** ✅ RESOLVIDO

---

## 1. Resumo Executivo

Este documento registra as correções aplicadas para garantir a validação do Google reCAPTCHA v2 nos formulários do sistema. Os ajustes foram realizados no back-end (Razor Pages / C#) e seguem o mesmo padrão já estabelecido para o formulário **Fale Conosco** (documentado separadamente).

O objetivo principal foi assegurar que, caso o usuário não interaja com o widget "Não sou um robô", o servidor bloqueie o avanço do formulário e exiba uma mensagem de alerta clara, sem depender exclusivamente da validação do lado do cliente.

---

## 2. Estratégia de Validação Adotada

A abordagem utilizada consiste em verificar, dentro do método `OnPostAsync` de cada página, se o token gerado pelo reCAPTCHA foi enviado junto à requisição. Se o campo `g-recaptcha-response` estiver ausente ou vazio, um erro é injetado manualmente no `ModelState`, impedindo o prosseguimento do fluxo.

```csharp
var recaptchaResponse = Request.Form["g-recaptcha-response"].ToString();
if (string.IsNullOrEmpty(recaptchaResponse))
{
    ModelState.AddModelError(string.Empty, "Por favor, marque a caixa 'Não sou um robô' para continuar.");
}
if (!ModelState.IsValid)
{
    return Page();
}
```

> **Observação:** O atributo `[ValidateRecaptcha]` deve permanecer presente no back-end do HTML correspondente a cada página corrigida.

---

## 3. Arquivos Modificados

### 3.1 `Ouvidoria/CadastroUsuario.cshtml.cs`

**Modificação:** Adicionada verificação de token reCAPTCHA no início do método `OnPostAsync`, antes da checagem do `ModelState.IsValid`.

A validação garante que o usuário tenha interagido com o widget antes de prosseguir com o cadastro.

---

### 3.2 `Ouvidoria/Login.cshtml.cs`

**Modificação:** Inserida a mesma verificação de token reCAPTCHA no método `OnPostAsync`.

O alerta passa a ser exibido na tela de login do módulo de Ouvidoria caso o campo de resposta do reCAPTCHA esteja vazio.

---

### 3.3 `Ouvidoria/RegistrarManifestacao.cshtml.cs`

**Modificação:** Adicionada verificação de token reCAPTCHA como **Passo 1** explícito dentro do método `OnPostAsync`, antes de qualquer outra lógica de negócio.

```csharp
// 1. VALIDAÇÃO DO RECAPTCHA
var recaptchaResponse = Request.Form["g-recaptcha-response"].ToString();
if (string.IsNullOrEmpty(recaptchaResponse))
{
    ModelState.AddModelError(string.Empty, "Por favor, marque a caixa 'Não sou um robô'.");
}
```

---

## 4. Matriz de Arquivos Modificados

| Arquivo | Modificação Realizada | Objetivo |
| :--- | :--- | :--- |
| `Ouvidoria/CadastroUsuario.cshtml.cs` | Verificação de `g-recaptcha-response` em `OnPostAsync` | Bloquear cadastro sem interação com reCAPTCHA |
| `Ouvidoria/Login.cshtml.cs` | Verificação de `g-recaptcha-response` em `OnPostAsync` | Bloquear login sem interação com reCAPTCHA |
| `Ouvidoria/RegistrarManifestacao.cshtml.cs` | Verificação de `g-recaptcha-response` como passo 1 em `OnPostAsync` | Bloquear registro de manifestação sem reCAPTCHA |

---

## 5. Padrão para Replicação em Outros Formulários

Para espelhar esta correção em outros formulários do sistema, basta aplicar o seguinte bloco no início do método `OnPostAsync`, **antes** de qualquer checagem de `ModelState.IsValid`:

```csharp
// VALIDAÇÃO DO RECAPTCHA
var recaptchaResponse = Request.Form["g-recaptcha-response"].ToString();
if (string.IsNullOrEmpty(recaptchaResponse))
{
    ModelState.AddModelError(string.Empty, "Por favor, marque a caixa 'Não sou um robô' para continuar.");
}
if (!ModelState.IsValid)
{
    return Page();
}
```

Verificar também se o atributo `[ValidateRecaptcha]` está presente na declaração do método `OnPostAsync` no arquivo `.cshtml.cs` correspondente.

---

## 6. Referências

- Relatório técnico do formulário **Fale Conosco** (depuração de `ModelState`, SMTP e reCAPTCHA) — documento separado no repositório.
- Documentação Google reCAPTCHA v2: [https://developers.google.com/recaptcha/docs/display](https://developers.google.com/recaptcha/docs/display)