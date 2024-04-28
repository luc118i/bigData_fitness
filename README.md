from selenium import webdriver
from selenium.webdriver.chrome.service import Service as ChromeService
from selenium.webdriver.common.by import By
from webdriver_manager.chrome import ChromeDriverManager
import time
import csv

def scrape_whey_data():
    # URL do site do Mercado Livre com os dados dos produtos de whey protein
    url = "https://lista.mercadolivre.com.br/whey-protein#D[A:Whey%20Protein]"

    headers = {'User-Agent': "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 \
        (KHTML, like Gecko) Chrome/123.0.0.0 Safari/537.36"
               }
    options = webdriver.ChromeOptions()
    options.add_argument('--headless')

    driver = webdriver.Chrome(service=ChromeService(ChromeDriverManager().install()), options=options)
    driver.get(url)
    time.sleep(2)

    # Aguarda até que os produtos sejam carregados (você pode ajustar o tempo conforme necessário)
    driver.implicitly_wait(10)

    # Encontra todos os elementos que contêm informações dos produtos
    products = driver.find_elements(By.CLASS_NAME, 'ui-search-layout__item')

    # Lista para armazenar os dados dos produtos
    whey_data = []

    # Loop pelos produtos e extrai as informações desejadas
    for product in products:
        # Extrai o nome do produto
        name = product.find_element(By.CLASS_NAME, 'ui-search-item__title').text.strip()

        # Extrai o preço do produto, se estiver disponível
        price_element = product.find_elements(By.CLASS_NAME, 'andes-money-amount__fraction')
        price = price_element[0].text.strip() if price_element else 'Preço não disponível'

        # Adiciona os dados do produto à lista
        whey_data.append({'Name': name, 'Price': price})

    # Fecha o navegador
    driver.quit()

    return whey_data


def save_to_csv(data, filename):
    # Escreve os dados em um arquivo CSV
    with open(filename, 'w', newline='', encoding='utf-8') as csvfile:
        fieldnames = ['Name', 'Price']
        writer = csv.DictWriter(csvfile, fieldnames=fieldnames)

        writer.writeheader()
        for item in data:
            writer.writerow(item)

if __name__ == "__main__":
    # Chama a função para raspagem dos dados
    whey_data = scrape_whey_data()

    # Se os dados foram obtidos com sucesso, salva-os em um arquivo CSV
    if whey_data:
        save_to_csv(whey_data, 'whey_data.csv')
        print("Dados salvos com sucesso em 'whey_data.csv'")
    else:
        print("Não foi possível obter os dados.")
