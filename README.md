# organograma
"howto" - explicando cada parte dos códigos, a estrutura dos arquivos e os requisitos para o funcionamento.

---

# **Documentação do Projeto de Captura de Organograma**

## **1. Visão Geral do Projeto**

Este projeto utiliza **Python** e a **biblioteca Selenium** para automatizar a navegação em um sistema de organograma, extraindo nomes de colaboradores por áreas de uma página web. O processo envolve realizar login no sistema, acessar a aba do organograma e capturar os dados de colaboradores, salvando-os em arquivos de texto.

### **Funcionalidades principais:**
- **Login automatizado** no sistema de organograma.
- **Navegação automatizada** pela estrutura hierárquica de áreas.
- **Captura de dados** (nomes de colaboradores) em cada área.
- **Geração de arquivos de texto** com a lista de áreas e respectivos colaboradores.

---

## **2. Estrutura dos Arquivos**

- **credenciais.py**: Script responsável por salvar e gerenciar as credenciais de login de forma segura, utilizando a biblioteca `keyring`.
  
- **login_script.py**: Script principal que realiza o login no sistema e chama o fluxo de automação para navegar pelo organograma.

- **workflow.py**: Script que implementa a lógica de navegação no organograma e a captura dos dados dos colaboradores.

---

## **3. Requisitos de Funcionamento**

### **Bibliotecas necessárias:**

- **Selenium**: Para automatizar a interação com o navegador.
  - Instalar via pip: `pip install selenium`
  
- **Keyring**: Para armazenamento seguro de credenciais.
  - Instalar via pip: `pip install keyring`
  
- **Webdriver**: Dependendo do navegador utilizado (Chrome, Firefox, etc.), é necessário o driver correspondente.
  - **ChromeDriver** para Google Chrome, por exemplo.
  
### **Passos para instalação e configuração:**

1. **Instalar as bibliotecas** citadas acima utilizando pip.
2. **Baixar e configurar o WebDriver** para o navegador desejado.
3. **Salvar as credenciais** utilizando o script `credenciais.py`.

---

## **4. Descrição dos Arquivos e Funções**

### **credenciais.py**:

Este script permite salvar as credenciais de login de forma segura.

**Funções principais**:

- `salvar_credenciais()`: Solicita ao usuário o nome de usuário e senha e os armazena no gerenciador de senhas (keyring).

---

### **login_script.py**:

Este script faz o login no sistema de organograma e inicia o processo de captura de dados chamando o script `workflow.py`.

**Principais partes do código**:
- **Login automático**: Utiliza o Selenium para simular o preenchimento dos campos de login e a submissão do formulário.
- **Navegação no sistema**: Após o login, redireciona para a página principal do workflow e chama o fluxo de automação para acessar o organograma.

```python
def realizar_login():
    usuario = keyring.get_password("expresso_login", "usuario")
    senha = keyring.get_password("expresso_login", "senha")

    driver = webdriver.Chrome()
    driver.get("https://expresso.pr.gov.br/login.php")
    
    # Preenchimento dos campos e submissão
    campo_usuario = driver.find_element(By.ID, "user")
    campo_senha = driver.find_element(By.ID, "passwd")
    
    # Digitação simulada
    for letra in usuario:
        campo_usuario.send_keys(letra)
        time.sleep(0.2)
    for letra in senha:
        campo_senha.send_keys(letra)
        time.sleep(0.2)
    
    campo_senha.send_keys(Keys.RETURN)  # Pressiona Enter
    
    # Após login, chama a função de workflow
    workflow.acessar_organograma(driver)
    workflow.buscar_areas(driver)

    return driver
```

---

### **workflow.py**:

Aqui ocorre a captura dos dados das áreas e dos colaboradores.

**Funções principais**:

1. **acessar_organograma(driver)**:
   - Navega até a aba do organograma.

2. **buscar_areas(driver)**:
   - Captura as áreas disponíveis no organograma e, em cada área, coleta os nomes dos colaboradores.
   - Utiliza o seletor CSS `#employeeList tbody tr td:first-child a` para capturar apenas o primeiro `<td>` (que contém o nome do colaborador).
   - Salva os nomes capturados no arquivo `lista_nomes.txt`.

```python
def buscar_areas(driver):
    with open("lista_areas.txt", "w") as arquivo_areas, open("lista_nomes.txt", "w") as arquivo_nomes:
        areas = driver.find_elements(By.CSS_SELECTOR, "div#orgchartAreas a")
        
        for area in areas:
            nome_area = area.text
            arquivo_areas.write(nome_area + "\n")
            
            try:
                area.click()  # Clica na área
                colaboradores = driver.find_elements(By.CSS_SELECTOR, "#employeeList tbody tr td:first-child a")
                
                for colaborador in colaboradores:
                    nome_colaborador = colaborador.text
                    if nome_colaborador:
                        arquivo_nomes.write(nome_colaborador + "\n")
            
            except Exception as e:
                print(f"Erro ao capturar dados da área {nome_area}: {e}")
```

---

## **5. Arquivos Gerados**

- **lista_areas.txt**: Contém a lista de todas as áreas acessadas durante a execução do script.
- **lista_nomes.txt**: Armazena os nomes dos colaboradores de cada área, organizados por área.

---

## **6. Execução do Projeto**

### **Passos para execução:**

1. **Salvar credenciais**:
   - Execute o script `credenciais.py` para inserir e salvar o nome de usuário e senha no keyring.

2. **Executar o script principal**:
   - Execute o `login_script.py`, que fará login automaticamente no sistema e iniciará a captura dos dados do organograma.

3. **Verificar os resultados**:
   - Após a execução, os arquivos `lista_areas.txt` e `lista_nomes.txt` conterão as áreas e os colaboradores extraídos.

---

## **7. Possíveis Erros e Soluções**

- **Erro de Timeout**: Aumentar o tempo de espera (`time.sleep`) antes de interagir com certos elementos.
- **Erro de Elemento não encontrado**: Garantir que o seletor CSS utilizado para capturar os elementos está correto e que os elementos estão visíveis na página.
