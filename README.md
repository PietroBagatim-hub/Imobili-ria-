<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Painel de Cadastro de Imóveis</title>
  <style>
    body { font-family: sans-serif; max-width: 700px; margin: 20px auto; padding: 20px; }
    .form-group { margin-bottom: 15px; }
    label { display: block; font-weight: bold; margin-bottom: 5px; }
    input, select, textarea { width: 100%; padding: 8px; box-sizing: border-box; }
    button { background: #007bff; color: white; padding: 12px 20px; border: none; cursor: pointer; font-size: 16px; }
  </style>
</head>
<body>
  <h2>Cadastrar Novo Imóvel</h2>
  <form id="formImovel">
    <div class="form-group">
      <label>Título do Anúncio:</label>
      <input type="text" id="titulo" required placeholder="Ex: Casa 4 Suítes em Condomínio Fechado">
    </div>

    <div class="form-group">
      <label>Finalidade:</label>
      <select id="finalidade">
        <option value="Venda">Venda</option>
        <option value="Locação">Locação</option>
        <option value="Venda e Locação">Venda e Locação</option>
      </select>
    </div>

    <div class="form-group">
      <label>Valor Pedido (R$):</label>
      <input type="number" id="valor" required placeholder="2500000">
    </div>

    <div class="form-group">
      <label>Dados Privados do Proprietário (Ocultos no Site):</label>
      <textarea id="proprietario" placeholder="Nome, Telefone, Matrícula do Imóvel, Inscrição IPTU"></textarea>
    </div>

    <div class="form-group">
      <label>Descrição Pública do Imóvel:</label>
      <textarea id="descricao" rows="4" required></textarea>
    </div>

    <div class="form-group">
      <label>Fotos em Alta Qualidade (Múltiplas):</label>
      <input type="file" id="fotos" multiple accept="image/*" required>
    </div>

    <button type="submit" id="btnSalvar">Salvar e Sincronizar</button>
  </form>

  <script>
    const CLOUD_NAME = "SEU_CLOUD_NAME_AQUI";
    const UPLOAD_PRESET = "SEU_PRESET_AQUI";
    const GH_TOKEN = "SEU_TOKEN_GITHUB_AQUI"; 
    const REPO_OWNER = "SEU_USUARIO_GITHUB";
    const REPO_NAME = "NOME_DO_REPOSITORIO";

    document.getElementById('formImovel').addEventListener('submit', async (e) => {
      e.preventDefault();
      const btn = document.getElementById('btnSalvar');
      btn.disabled = true;
      btn.innerText = "Enviando fotos e salvando...";

      try {
        const files = document.getElementById('fotos').files;
        const fotoUrls = [];

        // 1. Envio das fotos em alta qualidade para o Cloudinary
        for (let file of files) {
          const formData = new FormData();
          formData.append('file', file);
          formData.append('upload_preset', UPLOAD_PRESET);

          const res = await fetch(`https://api.cloudinary.com/v1_1/${CLOUD_NAME}/image/upload`, {
            method: 'POST',
            body: formData
          });
          const data = await res.json();
          fotoUrls.push(data.secure_url);
        }

        // 2. Montagem do Objeto do Imóvel
        const idImovel = `IMOVEL-${Date.now()}`;
        const imovelData = {
          id: idImovel,
          titulo: document.getElementById('titulo').value,
          finalidade: document.getElementById('finalidade').value,
          valor: parseFloat(document.getElementById('valor').value),
          descricao: document.getElementById('descricao').value,
          proprietario: document.getElementById('proprietario').value,
          fotos: fotoUrls,
          dataCadastro: new Date().toISOString()
        };

        // 3. Commit do JSON no GitHub via API
        const path = `data/imoveis/${idImovel}.json`;
        const content = btoa(unescape(encodeURIComponent(JSON.stringify(imovelData, null, 2))));

        await fetch(`https://api.github.com/repos/${REPO_OWNER}/${REPO_NAME}/contents/${path}`, {
          method: 'PUT',
          headers: {
            'Authorization': `token ${GH_TOKEN}`,
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({
            message: `Novo imóvel cadastrado: ${idImovel}`,
            content: content
          })
        });

        alert("Imóvel cadastrado com sucesso! O site e o catálogo do WhatsApp serão atualizados em instantes.");
        document.getElementById('formImovel').reset();
      } catch (err) {
        alert("Erro ao cadastrar imóvel: " + err.message);
      } finally {
        btn.disabled = false;
        btn.innerText = "Salvar e Sincronizar";
      }
    });
  </script>
</body>
</html>
