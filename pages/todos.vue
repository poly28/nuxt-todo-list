<template>
  <div>
    <header>
      <h2>Todo List</h2>
    </header>

    <!-- 入力フォームここから -->
    <form>
      <input
        id="Task"
        type="text"
        placeholder=" Put on your task!"
        v-model="newTask"
      />
      <button @click.prevent="addTask">Add</button>
    </form>
    <!-- 入力フォームここまで -->
    <hr />

    <!-- 追加機能ここから -->
    <section class="additional-function">
      <!-- 絞り込み機能ここから -->
      <nav>
        <h3>【Filter】</h3>
        <input
          type="checkbox"
          id="check-waiting"
          v-model="checkedStatus.Waiting"
        />
        <label for="check-waiting">Waiting</label>
        <br />

        <input type="checkbox" id="check-doing" v-model="checkedStatus.Doing" />
        <label for="check-doing">Doing</label>
        <br />

        <input
          type="checkbox"
          id="check-complete"
          v-model="checkedStatus.Complete"
        />
        <label for="check-complete">Complete</label>
        <br />
      </nav>
      <!-- 絞り込み機能ここまで -->
      <!-- ソート機能ここから -->
      <nav>
        <h3>【Sort】</h3>
        <select v-model.number="sortOrder">
          <option value="0"></option>
          <option value="1">ID昇順</option>
          <option value="2">ID降順</option>
          <option value="3">期限昇順</option>
          <option value="4">期限降順</option>
          <option value="5">ステータス昇順</option>
          <option value="6">ステータス降順</option>
        </select>
      </nav>
      <!-- ソート機能ここまで -->
    </section>
    <hr />
    <!-- Todoリストここから -->
    <table>
      <thead>
        <tr>
          <!-- テーブルの見出しレンダリング -->
          <th v-for="table in tableSet" :class="table.class" :key="table.id">
            {{ table.title }}
          </th>
          <!-- テーブルの見出しレンダリング -->
        </tr>
      </thead>

      <!-- タスクリストのレンダリング -->
      <tbody>
        <tr v-for="(todo, index) in filteredList" :key="todo.id">
          <td>{{ todo.id }}</td>
          <td class="td-task">
            <input v-if="isEdit" v-model="todo.task" />
            <template v-else>{{ todo.task }}</template>
          </td>

          <td class="td-detail">
            <input v-if="isEdit" v-model="todo.comment" />
            <template v-else>{{ todo.comment }}</template>
          </td>
          <!-- todo：min-dateに現在時刻を設定 -->
          <!-- min-data="limitStart" -->

          <td class="td-limit">
            <vue-ctk-date-time-picker
              v-if="isEdit"
              label="Select Limit!"
              formatted="YYYY/MM/DD HH:mm"
              format="YYYY/MM/DD HH:mm"
              v-model="todo.limit"
              overlay="true"
              minute-interval="30"
            ></vue-ctk-date-time-picker>
            <p v-else>{{ todo.limit }}</p>
          </td>

          <td>
            <select v-model="todo.status">
              <option value="Waiting">Waiting</option>
              <option value="Doing">Doing</option>
              <option value="Complete">Complete</option>
            </select>
          </td>
          <td>
            <button @click="rmTask(index)">×</button>
          </td>
          <td>
            <button v-if="!isEdit" @click="isEdit = !isEdit">Edit</button>
            <button v-else @click="isEdit = !isEdit">OK</button>
          </td>
        </tr>
      </tbody>
    </table>
    <!-- タスクリストのレンダリング -->

    <!-- 各種機能ここから -->
    <section>
      <!-- 全削除機能 -->
      <button @click="allDelete">All Delete</button>
    </section>
    <!-- 各種機能ここまで -->

    <!-- Todoリストここまで -->
    <hr />
  </div>
</template>

<script>
export default {
  data: function() {
    return {
      // タスクリストを格納するための配列
      todos: [],

      //ステータスによる絞り込み用配列
      checkedStatus: {
        Waiting: false,
        Doing: false,
        Complete: false
      },

      sortOrder: 0,

      // タスク追加用データ
      newTask: "",
      newDetail: "-",
      newLimit: "",
      isEdit: false,

      // テーブルの見出しレンダリング用
      tableSet: [
        { title: "ID", class: "th-id" },
        { title: "Task", class: "th-task" },
        { title: "Detail", class: "th-detail" },
        { title: "Limit", class: "th-limit" },
        { title: "Status", class: "th-status" },
        { title: "Delete", class: "th-delete" },
        { title: "Edit", class: "th-edit" }
      ]
    };
  },
  methods: {
    // タスク追加処理
    addTask() {
      // 入力フォームのバリデーション
      // タスク名の未入力検知
      if (this.newTask === "") {
        alert("タスク名が入力されていません。");
        return;
      }

      // タスク追加オブジェクト
      let newTask = {
        id: this.todos.length + 1,
        task: this.newTask,
        comment: this.newDetail,
        limit: this.newLimit,
        status: "Waiting"
      };
      // タスクの追加処理
      this.todos.push(newTask);

      // タスク追加用変数の初期化
      this.newTask = "";
      this.newDetail = "-";
      this.newLimit = "";
    },

    // タスク個別削除機能：タスク横の×を押して該当タスクを削除します
    rmTask(index) {
      this.todos.splice(index, 1);
    },

    // 全削除機能：All Deleteボタンを押してタスクを全削除します
    allDelete() {
      if (confirm("All delete OK?")) {
        this.todos = [];
      }
    }
  },
  computed: {
    //----------------------------------
    // タスクリスト レンダリング用フィルター
    //----------------------------------
    filteredList() {
      // フィルタリング実施後のリスト
      let newList = [];
      let i;

      // タスクリストフィルタリング処理
      for (i = 0; i < this.todos.length; i++) {
        let isShow = false;

        // ステータス：Waiting
        if (this.checkedStatus.Waiting) {
          // ステータスがWaitingのタスクをnewListに登録
          if (this.todos[i].status === "Waiting") {
            isShow = true;
          }
        }

        // ステータス：Doing
        if (this.checkedStatus.Doing) {
          // ステータスがDoingのタスクをnewListに登録
          if (this.todos[i].status === "Doing") {
            isShow = true;
          }
        }

        // ステータス：Complete
        if (this.checkedStatus.Complete) {
          // ステータスがCompleteのタスクをnewListに登録
          if (this.todos[i].status === "Complete") {
            isShow = true;
          }
        }

        // Filter未選択
        if (
          !this.checkedStatus.Waiting &&
          !this.checkedStatus.Doing &&
          !this.checkedStatus.Complete
        ) {
          isShow = true;
        }
        if (isShow) {
          newList.push(this.todos[i]);
        }
      }

      // ここにsort機能を記述
      switch (this.sortOrder) {
        case 0: // sort解除(初期値)
          break;
        case 1: // ID昇順
          newList.sort(function compareFunc(a, b) {
            return a.id - b.id;
          });
          break;
        case 2: // ID降順
          newList.sort(function compareFunc(a, b) {
            return b.id - a.id;
          });
          break;

        case 3: // 期限昇順
          newList.sort(function compareFunc(a, b) {
            if (a.limit > b.limit) return 1;
            if (a.limit < b.limit) return -1;
            return 0;
          });
          break;

        case 4: // 期限降順
          newList.sort(function compareFunc(a, b) {
            if (a.limit < b.limit) return 1;
            if (a.limit > b.limit) return -1;
            return 0;
          });
          break;

        case 5: // ステータス昇順
          newList.sort(function compareFunc(a, b) {
            let statusA = a.status.toUpperCase();
            let statusB = b.status.toUpperCase();
            if (statusA > statusB) return 1;
            if (statusA < statusB) return -1;
            return 0;
          });
          break;

        case 6: // ステータス降順
          newList.sort(function compareFunc(a, b) {
            let statusA = a.status.toUpperCase();
            let statusB = b.status.toUpperCase();
            if (statusA < statusB) return 1;
            if (statusA > statusB) return -1;
            return 0;
          });
          break;
      }

      // フィルタリングしたリストをリターン
      return newList;
    }
    // todo: 現在日時を返して過去日付の入力を防ぐ
    // limitStart() {
    // min-date に明日の9時を指定
    // const start = moment().add(1, 'days').hour(8);
    // return start.format('YYYY/MM/DD HH:mm');
    // const minDate = '2021/05/29 21:50';
    // return minDate;
    // return '2021/05/29 21:50';
    // },
  },
  watch: {
    // タスク削除時のIDの整合性を保つ
    todos() {
      for (let i = 0; i < this.todos.length; i++) {
        this.todos[i].id = i + 1;
      }
    }
  }
};
</script>

<style lang="scss" scoped>
/*================================
    ▽ 共通設定
================================*/
* {
  margin: 0;
}
body {
  background-color: rgb(245, 245, 245);
}
/*================================
    ▽ カラー
================================*/
$mainColor: rgb(102, 204, 255);
$headerBGC: $mainColor;
$tableHeadBGC: $mainColor;
$tableDataBGC: white;

/*================================
    ▽ ボーダー
================================*/
$taskBorder: 1px solid gray;
$tableBorder: 3px solid rgb(245, 245, 245);

/*================================
    ▽ 横幅
================================*/
// テーブルのヘッダー横幅
$idWidth: 50px;
$taskWidth: 150px;
$detailWidth: 200px;
$limitWidth: 200px;
$statusWidth: 100px;
$deleteWidth: 100px;
$editWidth: 50px;

/*================================
    ▽ フォントサイズ
================================*/
$titleFS: 28px;
$taskFS: 18px;
/*================================
    ▽ メインスタイル
================================*/
/*================================
    ▽ ヘッダー
================================*/
header {
  background-color: $headerBGC;
  color: white;
  height: 80px;
  line-height: 80px;
  padding: 0 40px;
  letter-spacing: 10px;
  font-size: $titleFS;
}

// 追加機能
.additional-function {
  display: flex;
}

// タスク入力フォーム
#Task {
  border: none;
  // border-bottom: $taskBorder;
  margin: 30px 0;
  margin-left: 40px;
  height: 28px;
  width: 300px;
  font-size: $taskFS;
  border-radius: 5px;
}

// タスクリスト
table {
  text-align: center;
  border-collapse: collapse;
  // margin-left: 30px;
  thead {
    tr {
      background-color: $tableHeadBGC;
      color: white;
      th {
        border: $tableBorder;
      }
      .th-id,
      .td-id {
        width: $idWidth;
      }
      .th-task,
      .td-task {
        width: $taskWidth;
      }
      .th-detail,
      .td-detail {
        width: $detailWidth;
      }
      .th-limit,
      .td-limit {
        width: $limitWidth;
      }
      .th-status,
      .td-status {
        width: $statusWidth;
      }
      .th-delete,
      .td-delete {
        width: $deleteWidth;
      }
      .th-edit.td-edit {
        width: $editWidth;
      }
    }
  }
  tbody {
    tr {
      td {
        background-color: $tableDataBGC;
        border: $tableBorder;
      }
    }
    .complete {
      background-color: gray;
      color: gray;
      text-decoration: line-through;
    }
  }
}
</style>
