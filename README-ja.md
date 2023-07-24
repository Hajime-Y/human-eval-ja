# HumanEval: Hand-Written Evaluation Set 

これは、以下の論文で説明されているHumanEval問題解決データセットの評価ハーネスです。
"[Evaluating Large Language Models Trained onCode](https://arxiv.org/abs/2107.03374)"

## Installation

必ずpython 3.7以降を使用してください:
```
$ conda create -n codex python=3.7
$ conda activate codex
```

このリポジトリをチェックアウトしてインストールする:
```
$ git clone https://github.com/Hajime-Y/human-eval-ja
$ pip install -e human-eval
```

## Usage

**このプログラムは、信頼されていないモデル生成コードを実行するために存在します。
堅牢なセキュリティ・サンドボックスの外では実行しないことが強く推奨されます。 `execution.py`の[execution
call](https://github.com/Hajime-Y/human-eval-ja/blob/312c5e5532f0e0470bf47f77a6243e02a61da530/human_eval/execution.py#L48-L58)は、潜在的に安全でない方法でコードを実行する前に、ユーザーにこの免責事項を読んでもらうために
意図的にコメントアウトされています。より詳しい情報と説明は `execution.py` のコメントを参照してください。**

上記の指示に従って実行できるようにした後、サンプルを生成し、以下のJSON Lines (jsonl)形式で保存します：
```
{"task_id": "Corresponding HumanEval task ID", "completion": "Completion only without the prompt"}
```
私たちは、`data` の下に `example_problem.jsonl` と `example_solutions.jsonl` を用意し、
書式の説明とデバッグの手助けをしています。

以下は、生成された補完を `samples.jsonl` に保存する、ほぼ機能的なサンプルコードです。
（動作させるには `generate_one_completion` を指定するだけです。）
```
from human_eval.data import write_jsonl, read_problems

problems = read_problems()

num_samples_per_task = 200
samples = [
    dict(task_id=task_id, completion=generate_one_completion(problems[task_id]["prompt"]))
    for task_id in problems
    for _ in range(num_samples_per_task)
]
write_jsonl("samples.jsonl", samples)
```

サンプルを評価するには以下を実行する
```
$ evaluate_functional_correctness samples.jsonl
Reading samples...
32800it [00:01, 23787.50it/s]
Running test suites...
100%|...| 32800/32800 [16:11<00:00, 33.76it/s]
Writing results to samples.jsonl_results.jsonl...
100%|...| 32800/32800 [00:00<00:00, 42876.84it/s]
{'pass@1': ..., 'pass@10': ..., 'pass@100': ...}
```
このスクリプトは、より詳細な情報を`<input_path>_results.jsonl`で終わる新しいファイルで提供する。
各行には、"passed"、"timed out"、"failed" のいずれかである実行結果 `result` と共に、
補完が `passed` されたかどうかが含まれるようになった。

簡単なチェックとして、サンプルの例では0.5 pass@1が得られるはずだ。
```
$ evaluate_functional_correctness data/example_samples.jsonl --problem_file=data/example_problem.jsonl
Reading samples...
6it [00:00, 3397.11it/s]
Running example suites...
100%|...| 6/6 [00:03<00:00,  1.96it/s]
Writing results to data/example_samples.jsonl_results.jsonl...
100%|...| 6/6 [00:00<00:00, 6148.50it/s]
{'pass@1': 0.4999999999999999}
```

Because there is no unbiased way of estimating pass@k when there are fewer
samples than k, the script does not evaluate pass@k for these cases. To
evaluate with other k values, pass `--k=<comma-separated-values-here>`. For
other options, see
kよりサンプル数が少ない場合のpass@kを推定する不偏の方法はないため、スクリプトはこのような場合の
pass@kを評価しません。k よりもサンプルが少ない場合、スクリプトはpass@kを評価しません。
他のk値で評価するには、`--k=<comma-separated-values-here>`を渡してください。
他のオプションについては以下を参照してください。
```
$ evaluate_functional_correctness --help
```
ただし、それ以外はデフォルト値を使用することをお勧めします。

## Known Issues

評価で使用するメモリはごくわずかですが、システムがRAM不足になると、次のようなエラーメッセージが
表示されることがあります。このような場合、正しいプログラムでも失敗することがあるため、メモリーを
解放して再試行することをお勧めします。
```
malloc: can't allocate region
```

## Citation

以下のbibtexエントリーを用いて引用してください：

```
@article{chen2021codex,
  title={Evaluating Large Language Models Trained on Code},
  author={Mark Chen and Jerry Tworek and Heewoo Jun and Qiming Yuan and Henrique Ponde de Oliveira Pinto and Jared Kaplan and Harri Edwards and Yuri Burda and Nicholas Joseph and Greg Brockman and Alex Ray and Raul Puri and Gretchen Krueger and Michael Petrov and Heidy Khlaaf and Girish Sastry and Pamela Mishkin and Brooke Chan and Scott Gray and Nick Ryder and Mikhail Pavlov and Alethea Power and Lukasz Kaiser and Mohammad Bavarian and Clemens Winter and Philippe Tillet and Felipe Petroski Such and Dave Cummings and Matthias Plappert and Fotios Chantzis and Elizabeth Barnes and Ariel Herbert-Voss and William Hebgen Guss and Alex Nichol and Alex Paino and Nikolas Tezak and Jie Tang and Igor Babuschkin and Suchir Balaji and Shantanu Jain and William Saunders and Christopher Hesse and Andrew N. Carr and Jan Leike and Josh Achiam and Vedant Misra and Evan Morikawa and Alec Radford and Matthew Knight and Miles Brundage and Mira Murati and Katie Mayer and Peter Welinder and Bob McGrew and Dario Amodei and Sam McCandlish and Ilya Sutskever and Wojciech Zaremba},
  year={2021},
  eprint={2107.03374},
  archivePrefix={arXiv},
  primaryClass={cs.LG}
}
```
