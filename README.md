# instr2questions

## Подготовка среды
1. Настроить conda environment: установить все зависимости, перечисленные в [requirements.txt](https://github.com/soyeonm/FILM/blob/public/requirements.txt) для FILM + библиотеку transformers.
2. [Скачать обученные BERTы](https://drive.google.com/file/d/1KQSpEBPd51x7tpF0rE1PVGCliRhuNhue/view). Поместить модели в папку **models** в рабочей директории.
3. Скачать [alfred_data](https://drive.google.com/file/d/1m0q7QYmmhSTOOoS62FdxSSXmQl2TY4gj/view?usp=sharing), поместить содержимое в папку **data/alfred_data** в рабочей директории.
4. Повторить предобработку данных, [как сделано в FILM](https://github.com/soyeonm/FILM#additional-preliminaries-to-use-alfred-scenes), чтобы получить отдельный json-файл для каждого задания:
   
   - Выполните команду
   ```
   $ export INSTR2QUES_ROOT=$(pwd)
   ```

   - Перейдите в директорию, куда хотите склонировать alfred:
   ```
   $ git clone https://github.com/askforalfred/alfred.git
   $ export ALFRED_ROOT=$(pwd)/alfred
   ```
   - Выполните команды
   ```
   $ cd $ALFRED_ROOT
   $ python models/train/train_seq2seq.py --data data/json_2.1.0 --model seq2seq_im_mask --dout exp/model:{model},name:pm_and_subgoals_01 --splits data/splits  /oct21.json --gpu --batch 8 --pm_aux_loss_wt 0.1 --subgoal_aux_loss_wt 0.1 --preprocess
   ```
   Это займёт 5-15 мин. Вывод должен быть таким:
   ![изображение](https://user-images.githubusercontent.com/64375679/170740281-29d4ec7a-e0ba-4425-9ba3-f3d9099832f8.png)
   Программа завершится с ошибкой, но на это можно не обращать внимания.
   
   - Выполните команды, чтобы скопировать файлы в рабочую директорию проекта
   ```
   $ cp -r data/json_2.1.0 $INSTR2QUES_ROOT/json_2.1.0
   $ cp -r data/splits/oct21.json $INSTR2QUES_ROOT/splits/oct21.json
   ```
   

## Запуск

Модуль работает в два этапа:
- [end_to_end_outputs.py](end_to_end_outputs.py) получает выходы BERTов. Результаты (тип задания, предсказанные аргументы шаблонов) для всех возможных инструкций из выборки записываются в отдельный выходной файл.

  Пример команды запуска:
  ```
  python end_to_end_outputs.py -sp tests_seen -m models -o tests_seen --no_appended
  ```
  -sp - часть датасета (tests_seen, tests_unseen, valid_seen и т.д.) \
  -m - папка с обученными моделями \
  -o - выходной pickled-файл \
  --no_appended - параметр нужен, чтобы не записывать low_step инструкции в выходной файл (чтобы обращаться к заданиям только через глобальную цель).

- [tasks_to_questions.py](tasks_to_questions.py) получает список подзадач и соответствующих вопросов. Результаты записывает в выходной json-файл.

  Пример команды запуска:
  ```
  python tasks_to_questions.py  --path_to_pickled_data test_unseen.p --split tests_unseen
  ```
  --path_to_pickled_data - путь к файлу, полученному на предыдущем шаге \
  --split - часть датасета (tests_seen, tests_unseen, valid_seen и т.д.)
  
  Пример выходного файла: [tests_unseen_questions.json](tests_unseen_questions.json)
