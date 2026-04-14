# Pixelmon EV Extractor & Data

MinecraftのポケモンMod「Pixelmon」の本体ファイル（.jar）から、全ポケモンの「努力値（EVs）」データを直接抽出し、一覧化したデータセットです。

## 収録データ

* `pixelmon_ev_list.csv`: 全1026系統の努力値一覧データ（CSV形式）

  * AI（ChatGPTやClaude等）にアップロードすることで、「特攻が2以上で、素早さも稼げるポケモン」といった高度な育成ターゲットの検索や、スプレッドシートでの独自解析が可能になります。

## 解析に使用したコード（参考）

本リポジトリのCSVデータを生成するために、Rocky Linux環境で実際に使用したコードです。ご自身の環境で検証・再現する際にご活用ください。

### 1. jarファイルからのデータ抽出（ワンライナー）

Modが配置されているディレクトリにて実行し、生のJSONデータから努力値を持つフォルムだけを抽出します。

```bash
python3 -c "
import zipfile, json
z = zipfile.ZipFile('Pixelmon-1.21.1-9.3.14-universal.jar')
for f in z.namelist():
    if f.startswith('data/pixelmon/species/') and f.endswith('.json'):
        try:
            data = json.loads(z.read(f))
            for form in data.get('forms', []):
                evs = form.get('evYields')
                if evs:
                    print(f'{data[\"name\"]}({form.get(\"name\", \"base\")}): {evs}')
        except: continue
" > ev_evidence_log.txt
```

### 2. CSVへの構造化スクリプト (`convert_ev.py`)

抽出したログファイル (`ev_evidence_log.txt`) を読み込み、扱いやすいCSV形式に変換します。

```python
import re
import csv

input_file = 'ev_evidence_log.txt'
output_file = 'pixelmon_ev_list.csv'
ev_types = ['hp', 'attack', 'defense', 'specialAttack', 'specialDefense', 'speed']

with open(input_file, 'r', encoding='utf-8') as f, \
     open(output_file, 'w', newline='', encoding='utf-8') as csvfile:
    
    writer = csv.writer(csvfile)
    writer.writerow(['Pokemon', 'Form'] + ev_types)
    
    for line in f:
        match = re.match(r"(.+)\((.+)\): \{(.+)\}", line.strip())
        if match:
            name, form, ev_data_raw = match.groups()
            row = [name, form]
            for etype in ev_types:
                val_match = re.search(f"'{etype}': (\\d+)", ev_data_raw)
                row.append(int(val_match.group(1)) if val_match else 0)
            writer.writerow(row)

print(f"変換完了: {output_file}")
```

## 関連記事

本データの抽出プロセスや、エビデンスに基づく育成手法については、以下のブログ記事で詳しく解説しています。

* [【Pixelmon.jarの解析記録】データから導き出す「確実な努力値稼ぎ」のエビデンス](https://haruuublog.com/blog/game-modding/ev-evidence)
