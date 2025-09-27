# ControleDeEstoque
 Controle de estoque desenvolvido utilizando a linguagem de programação C# (Windows Forms), a ferramenta de desenvolvimento Visual Studio e o Banco de dados SQL Server. O sistema utiliza o conceito de camadas (Três camadas) e terá cadastro de clientes, fornecedores, categorias, subcategorias, produtos, tipo de pagamento, unidade de medida, compra, venda, rotinas de backup.
 
Caso queira desenvolver o sistema do zero basta clicar no link: https://www.youtube.com/playlist?list=PLfvOpw8k80Wqj1a66Qsjh8jj4hlkzKSjA

Caso queira aprender muito mais acesse:
- dfilitto (site): https://dfilitto.com.br/
- dfilitto (Udemy): https://www.udemy.com/user/danilo-filitto/
- dfilitto (YouTube): https://www.youtube.com/danilofilittoppr

## Funcionalidades do sistema
### Cadastros
- Clientes
- Fornecedores
- Produtos
- Tipo de pagamento
- Unidade de medida
- Categoria
- Subcategoria
### Consultas
- Clientes
- Fornecedores
- Produtos
- Tipo de pagamento
- Unidade de medida
- Categoria
- Subcategoria
### Movimentações
- Compra
- Venda
### Ferramentas
- Backup da base de dados



📘 Documento: Passo a passo para sistema de controle de estoque com leitura de PDFs e IA (Java + React)
🧾 Controle de Estoque com Leitura de PDFs e IA

Objetivo: Criar um sistema onde o usuário envia um PDF (nota fiscal, recibo, foto ou scan) e o sistema reconhece automaticamente os produtos comprados, valores, e registra a entrada no estoque.

🧱 Tecnologias Utilizadas
Camada	Tecnologia
Frontend	React.js
Backend	Java (Spring Boot)
OCR	Tesseract (via Tess4J)
Leitura PDF	Apache PDFBox
IA	OpenAI GPT-3.5 Turbo (API)
Comunicação	REST API (JSON)
🗂️ Estrutura Geral do Projeto
- frontend/ (React)
- backend/ (Spring Boot)
  ├── controller/
  ├── service/
  ├── util/
      ├── PdfUtil.java
      ├── OcrUtil.java
      ├── OpenAIUtil.java

🔄 Fluxo Completo

Usuário envia um PDF via formulário no React.

Backend Java recebe o arquivo.

Verifica se o PDF é texto ou imagem:

Se for texto → extrai com PDFBox

Se for imagem ou scan → extrai com OCR (Tess4J)

Texto extraído é enviado à API da OpenAI

OpenAI responde com um JSON estruturado com:

Fornecedor

Data

Itens (nome, quantidade, valor)

Valor total

Backend envia esse JSON para o frontend.

Frontend exibe os dados para revisão e confirmação.

Se aprovado, o sistema registra a movimentação de estoque.

🔧 Etapas Técnicas
1. Upload do PDF no React
const formData = new FormData();
formData.append('file', selectedFile);

await fetch('http://localhost:8080/api/upload', {
  method: 'POST',
  body: formData,
});

2. Backend Java - Upload Controller
@PostMapping("/api/upload")
public ResponseEntity<?> uploadPdf(@RequestParam("file") MultipartFile file) throws Exception {
    File tempFile = File.createTempFile("uploaded", ".pdf");
    file.transferTo(tempFile);

    String extractedText = PdfUtil.extractTextFromPdf(tempFile);
    
    if (extractedText.trim().isEmpty()) {
        List<File> images = PdfUtil.convertPdfToImages(tempFile);
        StringBuilder ocrText = new StringBuilder();
        for (File img : images) {
            ocrText.append(OcrUtil.doOCR(img));
        }
        extractedText = ocrText.toString();
    }

    String structuredJson = OpenAIUtil.sendToChatGPT(extractedText);
    return ResponseEntity.ok(structuredJson);
}

3. Leitura de PDF com Apache PDFBox
public class PdfUtil {
    public static String extractTextFromPdf(File file) throws IOException {
        try (PDDocument document = PDDocument.load(file)) {
            PDFTextStripper stripper = new PDFTextStripper();
            return stripper.getText(document);
        }
    }

    public static List<File> convertPdfToImages(File pdfFile) throws IOException {
        PDDocument document = PDDocument.load(pdfFile);
        PDFRenderer renderer = new PDFRenderer(document);
        List<File> images = new ArrayList<>();

        for (int i = 0; i < document.getNumberOfPages(); i++) {
            BufferedImage bim = renderer.renderImageWithDPI(i, 300);
            File img = File.createTempFile("page_" + i, ".png");
            ImageIO.write(bim, "png", img);
            images.add(img);
        }

        document.close();
        return images;
    }
}

4. OCR com Tess4J
public class OcrUtil {
    public static String doOCR(File imageFile) throws TesseractException {
        Tesseract tesseract = new Tesseract();
        tesseract.setDatapath("tessdata"); // caminho dos dados do idioma
        tesseract.setLanguage("por");
        return tesseract.doOCR(imageFile);
    }
}

5. Enviando texto para a OpenAI (gpt-3.5-turbo)
public class OpenAIUtil {
    private static final String API_KEY = "sua-chave-aqui";
    private static final String API_URL = "https://api.openai.com/v1/chat/completions";

    public static String sendToChatGPT(String extractedText) throws IOException {
        OkHttpClient client = new OkHttpClient();

        String prompt = "Extraia os dados da nota fiscal abaixo e retorne um JSON estruturado com fornecedor, data, produtos (nome, quantidade, valor unitário) e valor total:\n\n" + extractedText;

        String body = "{ \"model\": \"gpt-3.5-turbo\", \"messages\": [ {\"role\": \"user\", \"content\": \"" + prompt.replace("\"", "\\\"") + "\"} ] }";

        Request request = new Request.Builder()
                .url(API_URL)
                .header("Authorization", "Bearer " + API_KEY)
                .post(RequestBody.create(body, MediaType.parse("application/json")))
                .build();

        try (Response response = client.newCall(request).execute()) {
            if (!response.isSuccessful()) throw new IOException("Erro: " + response);
            return response.body().string();
        }
    }
}

✅ Resultados esperados
Exemplo de JSON de retorno:
{
  "fornecedor": "Comercial ABC",
  "data": "2025-09-25",
  "itens": [
    {
      "produto": "Parafuso 10mm",
      "quantidade": 100,
      "valor_unitario": 0.50
    },
    {
      "produto": "Arruela 8mm",
      "quantidade": 200,
      "valor_unitario": 0.10
    }
  ],
  "valor_total": 70.00
}

🧪 Dicas de melhoria

Fazer correção automática de OCR com IA (ex: "Parafuzo" → "Parafuso")

Usar cache de PDFs processados para evitar retrabalho

Adicionar histórico de PDFs e movimentações

💰 Custos

Tess4J + Tesseract: 100% gratuito

Apache PDFBox: gratuito

OpenAI GPT-3.5 API: ~$0.001 por chamada (~750 palavras)

Alternativas Open Source de IA: Hugging Face (para uso local)