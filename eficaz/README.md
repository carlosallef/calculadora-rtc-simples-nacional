# Fork da Eficaz

Fork de [ReceitaFederal/calculadora-rtc-simples-nacional](https://github.com/ReceitaFederal/calculadora-rtc-regime-regular)
(licença MIT, preservada em `LICENSE`). A branch `main` acompanha a da Receita e **não é usada** para build.

- A branch `eficaz` guarda só o que é nosso: `eficaz/Dockerfile`, `eficaz/VERSAO` e o workflow `eficaz-imagem.yml`.
- O build faz checkout da **tag `rtc-<versão>`** (o código da Receita intacto), confere que ela aponta para o commit
  registrado no ato oficial (Portaria Cotec 342/2026 para a 1.6.1) e publica `ghcr.io/carlosallef/rtc-simples:<versão>`.
- Trocar de versão: conferir o novo commit contra o ato, criar a tag `rtc-<versão>` nele e editar `eficaz/VERSAO`.
- Imagem: JRE 21, perfil `offline`, HTTP 8082 (`/api`), gestão 9103 (`/health`, `/metrics`). Sem autenticação:
  **nunca publicar porta para a internet**.
- **Banco:** este repo não tem flyway. O workflow faz checkout de `carlosallef/calculadora-rtc-regime-regular` na tag
  de `DB_TAG` (conferida contra `DB_COMMIT`) e gera o mesmo `calculadora-pro.db` do regular. `DB_DOWNLOAD_URL` não é usado.
