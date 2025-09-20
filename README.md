# Papers-compare-project-by-langflow
這次要做的是使用langflow作一個論文回覆聊天機器人，找尋不同的論文，當使用者輸入想要找的論文主題並且說明他想執行的動作 ex.比較論文書寫方式、對專有名詞的解釋差異 <br>
[最新更新 2024/12](#最新更新)
## 簡單的pdf回答機器人
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/simple_pdf.png)
上面的例子是一個簡單的pdf答覆聊天機器人，我們將有關一家店的所有資訊使用file以data形式output接到split text讓data轉成text chuncks. <br>
再將這些chuncks丟入vector store. <br>
### 什麼是vector stores，為甚麼要用vector stores
Vector Stores的用途是儲存和檢索由文本或其他資料產生的向量。在自然語言處理中，當你使用嵌入模型(這個例子使用OpenAI embedding)將文本轉換為向量，你需要高效的方式儲存這些向量並在需要時進行檢索。<br>
拿我的主題當作例子，當我將論文的PDF檔案轉換成向量後存入Vector DB，可以方便user查詢時根據內容相似性檢索相關的文章或段落。 <br>
以下是步驟:<br>
1. 解析PDF檔案的內容，提取出文本資料。
2. 將提取的文本通過一個嵌入模型轉換為向量。這些向量是文本的數學表示，能夠捕捉文本的語義信息。
3. 向量存入Vector DB，每篇文章或每個段落對應一個向量，可以在向量中保留相關的元數據，比如文章標題、作者、日期等。
4. 當用戶輸入查詢，查詢的輸入文字也會被轉換為向量。
5. 這個查詢向量將在Vector DB中進行比對，找到最相似的向量，並返回相關的文章或段落。
6. Vector DB會根據相似性返回最相關的結果，這些結果就是符合用戶查詢主題的文章。
<br>
然後我們將Astra DB回傳的data用parse data轉成text，接上prompt，使它成為聊天機器人可用的店家資訊，回答使用者的問題。<br>
參考連結:https://www.youtube.com/watch?v=rz40ukZ3krQ <br>

## 開始實作-論文答覆機器人version1
架構想法:`input`-`兩個prompt分別處理 1.使用者想找的論文 2.使用者想針對這些論文作的動作`-`將論文存進Astra DB`-`將資料傳給prompt 讓機器人去分析那些文章是使用者要的`-`機器人執行第二個prompt提出的動作ex.compare`
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/project_ver1.png)
我遇到的問題: <br>
### 一個詳細且考慮所有狀況的prompt? 還是input與prompt互相配合?
原先，我很直覺的想讓OpenAI的model直接從使用者的一串句子中判斷並提取出使用者可能要的論文關鍵字以及想執行的動作，在prompt中加入few-shot learning的方法，以為這樣就能解決。<br>
原本的prompt:<br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/poor_prompt.png) <br>
原本的input:<br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/poor_input.png) <br>
原本的output:<br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/poor_output.png) <br>
可以發現這違反我們的意圖，但在model不完善的情況下這是可預期的結果，使用者的輸入可能每次的風格、格式、提問方法都不一樣，這造成了難以讀取。<br>
所以我決定:`讓使用者輸入是有格式的`，方便model讀取、提高正確率。<br>
修改後的prompt:<br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/promote_prompt.png) <br>
修改後的input:<br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/promote_input.png) <br>
修改後的output:<br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/promote_output.png) <br>
### 手動下載論文放入vector DB還是上網爬?
我的目標是想使用者輸入想要的論文關鍵字之後，app去能下載到或預覽到論文的網站直接搜索(呼叫API)，但最後version1決定先從手動下載pdf開始。<br>
先將測試用的論文利用前面提到的簡單答覆機器人的方法丟入vector DB中當作model可使用的資料，Astra DB的search input跟OpenAI分析出使用者的article相連得出我們要的資料，再用prompt告訴model從這些資料找出user要的<br>
<br>
user's input:<br>
```
Article [ReAct], Do [the definition of zero shoot in different papers]
```
Astra DB搜索結果:<br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/ver1_component_output.png) <br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/ver1_component_text.png) <br>
圖中對應到論文的段落:<br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/ver1_pdf_result.png) <br>
prompt:<br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/ver1_find_pdf_prompt.png) <br>
OpenAI model輸出結果:<br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/ver1_chatgptaboutpdf_output.png) <br>

結果似乎還不是我想要的，我預期是它能告訴我是哪個論文以及提取整個論文出來並準備接收使用者想對論文們做的動作(version1中尚未實作)。<br>
我們試圖在parse data template加入{file_path}試試?<br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/ver1.5_parsedata_prompt.png)<br>
new prompt:<br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/ver1.5_prompt.png)<br>
result:<br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/ver1.5_chatgptaboutpdf.png)<br>
我們又更進一步了，得出的結果更精準了，但未來還可以作改進。<br>
## 使用爬蟲?
能不能不用手動下載論文，未來這題目的方向會試圖解決爬蟲下載論文的問題。<br>
參考資料(NLP - 用Selenium爬蟲『博碩士論文加值系統』):https://hackmd.io/@bessyhuang/H1dgAM3OI#%E7%88%AC%E8%9F%B2%E7%A8%8B%E5%BC%8F%E7%A2%BC%E6%99%BA%E8%83%BD%E7%89%88---%E8%AD%BD%E9%8C%9A <br>
實作(用於custom component):<br>
```python
# from langflow.field_typing import Data
from langflow.custom import Component
from langflow.io import MessageTextInput, Output
from langflow.schema import Data
from bs4 import BeautifulSoup
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait

class CustomComponent(Component):
    display_name = "Custom Component"
    description = "Use as a template to create your own component."
    documentation: str = "http://docs.langflow.org/components/custom"
    icon = "custom_components"
    name = "CustomComponent"

    inputs = [
        MessageTextInput(name="input_value", display_name="Input Value", value="Hello, World!"),
    ]

    outputs = [
        Output(display_name="Output", name="output", method="build_output"),
    ]

    def build_output(self) -> Data:
        browser = webdriver.Firefox()
        browser.get('https://ndltd.ncl.edu.tw/cgi-bin/gs32/gsweb.cgi/login?o=dwebmge')

        browser.find_element(By.ID,"ysearchinput0").send_keys(self.input_value)

        browser.find_element(By.ID, "gs32search").click()
        driver_wait = WebDriverWait(browser, 20, 0.5)

        html = browser.page_source

        soup = BeautifulSoup(html, 'html.parser')
        
        data = Data(value=soup.text)
        self.status = data
        return data
```
可能架構(未完成):<br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/project_may1.png) <br> 
input:<br> 
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/may1_input.png) <br> 
part of custom component output:<br> 
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/may1_component_text.png) <br>
output website:<br> 
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/may1_website.png) <br>
但這樣仍沒處理下載論文的問題，之後須再研究。<br>
## 使用agent?
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/agent_first_test.png) <br> 
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/ReAct.png) <br> 
prompt: <br>
```python
You run in a "loop" of Thought, Action, Observation.
At the end of the loop you output an Answer
Use Thought to describe your thoughts about the question you have been asked.
Use Action to run one of the actions available to you。
Observation will be the result of running those actions.

Example:

Question: Can you tell me who is the Joe Biden?
Thought: First I need to get the data of Joe Biden.
Action: Use Wikipedia get the information, search: Joe Biden.
Observation: Joe Biden's information.

Thought: With the information I get, I can simply introduce Joe Biden.
Action: Summarize the information.
Observation: Joseph Robinette Biden Jr. (born November 20, 1942) is the 46th and current president of the United States, serving since 2021. A member of the Democratic Party, he was the 47th vice president under President Barack Obama from 2009 to 2017 and represented Delaware in the U.S. Senate from 1973 to 2009.

Thought: I’ve finished.
Action: Output the answer.

Answer: Joseph Robinette Biden Jr. (born November 20, 1942) is the 46th and current president of the United States, serving since 2021. A member of the Democratic Party, he was the 47th vice president under President Barack Obama from 2009 to 2017 and represented Delaware in the U.S. Senate from 1973 to 2009.

And here is the user's question: {input}
Please follow the example, let me know the way you deal with question.
Remember, each loop should have Thought, Action, and Observation except final loop.
```
```python
Which singer won the Record of the Year in 66th Annual Grammy Awards?
```
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/agent_try.png) <br> 
## 未來方向
1. 研究agent的使用，使解析input更加準確
2. 研究如何將論文爬下來而不是手動下載
3. 找到優化vector DB分析資料的方法
4. 如何從vector DB將論文一整篇截取下來
5. 更好的架構
## 10/8問題
1. 在version1中，我可能有什麼方法讓model同時輸出pdf的位置、標題及文章所有內容，且文章不只一個
2. 我可能有什麼方法在langflow這個應用中不用手動去網站找論文下載而是透過使用API或者爬蟲方式將論文下載下來，這樣不只省時且可以不占空間

## 最新更新
![image]() <br>
最終報告之成果圖(2024/12/23) <br>
[報告Canva ppt連結](https://www.canva.com/design/DAGZ1mR5Bwk/sOjeuCkvebvJMkOiYj5c8w/view?utm_content=DAGZ1mR5Bwk&utm_campaign=designshare&utm_medium=link2&utm_source=uniquelinks&utlId=h7218b41ca3) <br>
已解決以下問題：<br>
- 論文皆使用爬蟲方式並提取文本
- 回答精準度提升
- 可同時比較之論文數增至五篇
- 可限制論文發行時間

架構圖：<br>
![image]() <br>

### Get papers' keyword and date
寒假更新(2025/01)改為直接提取文字而非使用AI分析<br>
結果相同<br>
![image]() <br>
```python
class CustomComponent(Component):
    display_name = "Extract Article Component"
    description = "Extracts content within 'Article' brackets from a given string."
    documentation: str = "http://docs.langflow.org/components/custom"
    icon = "custom_components"
    name = "ExtractArticleComponent"

    inputs = [
        MessageTextInput(name="input_value", display_name="Input Value", value="Article [AI], Date[from 2014-05 to 2024-05], Do [What can AI do mentioned in different articles]"),
    ]

    outputs = [
        Output(display_name="Extracted Article", name="output", method="build_output"),
    ]

    def build_output(self) -> Data:
        input_text = self.input_value
        match = re.search(r"Article \[(.*?)\]", input_text)  # 匹配 "Article [ ]"
        if match:
            extracted_content = match.group(1)
        else:
            extracted_content = "Not Found"

        data = Data(value=extracted_content)
        self.status = data
        return data
```

### Scrapying
```python
class CustomComponent(Component):
    display_name = "Custom Component"
    description = "Use as a template to create your own component."
    documentation: str = "http://docs.langflow.org/components/custom"
    icon = "custom_components"
    name = "Custom_Component"

    inputs = [
        MessageTextInput(name="article", display_name="Article", value="Hello, World!"),
        MessageTextInput(name="date", display_name="Date", value="Hello, World!"),
    ]

    outputs = [
        Output(display_name="Output", name="output", method="build_output_output"),
    ]

    def build_output_output(self) -> Data:
        browser = webdriver.Firefox()
        browser.get('https://arxiv.org/search/advanced')
        D = self.date
        match = re.search(r"'value':\s*'(.*?)'", D)
        extracted_value = match.group(1)
        dates = extracted_value.split(",")
        start_date = dates[0]
        end_date = dates[1]
        time.sleep(10)
        browser.find_element(By.ID, "date-filter_by-3").click()
        browser.find_element(By.ID, "date-from_date").send_keys(start_date)
        browser.find_element(By.ID, "date-to_date").send_keys(end_date)

        browser.find_element(By.ID, "terms-0-term").send_keys(re.search(r"'value':\s*'(.*?)'", self.article).group(1))
        time.sleep(2)
        browser.find_element(By.ID, "terms-0-term").send_keys(Keys.ENTER)
        
        time.sleep(5)
        links = browser.find_elements(By.PARTIAL_LINK_TEXT, "arXiv:")
        links[0].click()
        time.sleep(2)
        browser.find_element(By.XPATH, "//li[1]/a[contains(@href, '/pdf/')]").click()
        pdf_url = browser.current_url
        response = requests.get(pdf_url)
        pdf_data = response.content
        
        text_content = ""
        with fitz.open(stream=pdf_data, filetype="pdf") as pdf_document:
            for page_num in range(pdf_document.page_count):
                page = pdf_document[page_num]
                text_content += page.get_text()
        
        data = Data(value=text_content)
        browser.quit()
        self.done_status = 1
        return data
```

### Get the main points of papers related to the user’s request
寒假更新(2025/01)prompt更新
```
You are now a chatbot responsible for organizing a research paper.
You will receive a paper and a task.
While this task was originally designed for multiple papers, you should treat it as specifically applying to the paper you receive.
Please help extract passages or sentences from the paper that could potentially address this task - the more the better.
Don't worry about word count limits, as another model will later handle the work of comparing different papers.

Here is the task: {task}
and here is the paper: {paper}
```
### Final Reply
論文減縮至處理四篇<br>
```
There are some articles and the task that user requests.
Please "compare" these articles and finish the task.
As detailed as you can.
Imagine you are a postgraduate or a PhD, you need to get sufficient information from this task to study.

This is the article_1 : {a1}
This is the article_2 : {a2}
This is the article_3 : {a3}
This is the article_4 : {a4}

And finally, this is the task : {task}
```

### Why use
- **efficiency**：不需要逐篇閱讀論文，就能獲得想要的答案
- **without download**：不需要上網搜尋論文、下載，再上傳到聊天機器人。這樣不僅節省時間，也能節省儲存空間
- **compare**：系統可以同時比較五篇論文(後改為4篇)，包含不同年份的研究，並且具備未來功能擴充的潛力
- **give you ideas**：你可以向這個聊天機器人詢問某一領域的研究進展，或是特定術語的解釋。由於這些論文可能來自不同領域的研究者，你能從多個角度理解同一主題或術語。跨領域的分析甚至能激發新的想法

### Problems & Future
- sequential architecture可以確保每個網頁都能正確處理，避免出現預期外的錯誤。然而，這樣的處理時間可能仍然過長。我們需要開發一種方法，在保持執行品質的同時，讓架構能夠平行化，進一步優化使用者體驗。這是我目前在此專案中看到的最大挑戰。當我們嘗試做在地化時，模型的效能可能不如OpenAI，這必然會增加處理時間。如果耗時過長，使用者自然不會優先選擇這個工具
- 未來我們可以嘗試使用Agent來處理使用者的問題，讓使用者不必依照特定的輸入格式。Agent 可以自動分析問題並提取關鍵詞
- 若能提供更多可自訂化的搜尋選項會更有幫助，例如類似 ARXIV 的進階搜尋，允許使用多個關鍵字進行查詢
