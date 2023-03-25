---
title:  "Drift 마이그레이션"
categories:
- Flutter

toc: true
toc_sticky: true
toc_label: "Contents"
---


# Flutter drift migration

---

> drift로 개발한 어플리케이션의 사용자 피드백이 많이 들어와서 migration을 진행하게 되어 drift migration을 공부하면서 정리하려고 한다.
>

우선 마이그레이션이 무슨 뜻인지 알아보고, flutter drift에서는 어떻게 migration을 적용하는지 자세히 알아보자.

### Migration

> 데이터베이스에서 데이터 마이그레이션이란, 데이터베이스 스키마의 버전을 관리하기 위한 방법이다.
운영 환경에서는 데이터베이스의 변경이 있을 때(테이블 구조 변경, 데이터 전환, 테이블 생성 등 ) 마이그레이션을 진행하게 된다.
>

---

### Flutter drift Migrations

앱의 버전이 올라가면서, drift database의 테이블의 변경은 불가피하다.
: 새로운 기능이 추가될 시 기존 테이블의 컬럼 추가, 테이블 추가 혹은 컬럼을 변경하거나 제가하게 될 수도 있다.

이렇게 데이터베이스의 변경이 필요할 때는 drift의 migration api를 사용해 기존 버전의 데이터베이스를 최신 버전으로 업데이트 해야 한다.

drift는 migration을 쉽게할 수 있는 APIs를 제공한다.

### Basic

Drift는 Database 클래스의 schemaVersion을 변경한 후 데이터베이스를 변경할 수 있는 migration API를 제공한다.
이를 사용하기 위해서는 Database 클래스 내의 migration을 getter로 override하면 된다.

> 글로 보면 이해가 되지 않으니 공식문서에서 제공해주는 예를 보자.
>

- Todos 테이블(버전 2)의 dueDate 컬럼을 새로 추가하고 싶다면
  기본 테이블의 컬럼을 추가해준다.

```dart
class Todos extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text().withLength(min: 6, max: 10)();
  TextColumn get content => text().named('body')();
  IntColumn get category => integer().nullable()();
  DateTimeColumn get dueDate =>
      dateTime().nullable()(); // new, added column in v2
  IntColumn get priority => integer().nullable()(); // new, added column in v3
}
```

- migrtaion을 위해서는 schemaVersion을 순차적으로 올리면 된다.
- onUpgrade에서 from은 변경 전 버전, to는 변경 후 버전을 뜻한다.
- 버전이 2 보다 작다면 todos 테이블에 dueDate 컬럼을 추가하고
  버전이 3 보다 작으면 todos 테이블에 priority 컬럼을 추가하는 것을 알 수 있따.

```dart
@override
int get schemaVersion => 3; // bump because the tables have changed.

@override
MigrationStrategy get migration {
  return MigrationStrategy(
    onCreate: (Migrator m) async {
      await m.createAll();
    },
    onUpgrade: (Migrator m, int from, int to) async {
      if (from < 2) {
        // we added the dueDate property in the change from version 1 to
        // version 2
        await m.addColumn(todos, todos.dueDate);
      }
      if (from < 3) {
        // we added the priority property in the change from version 1 or 2
        // to version 3
        await m.addColumn(todos, todos.priority);
      }
    },
  );
}
// The rest of the class can stay the same
```

migration api를 사용하면 테이블을 생성하거나 삭제할 수도 있다.

- migration callback 안에서 “select”, “update”, “delete” 등의 작업도 수행할 수 있다.

  주의할 점은 예를 들어 특정 테이블의 컬럼을 추가할 때 해당 작업이 완료되지 전에는 “select”과 같은 명령어를 실행하면 안된다.
  일반적으로 “select”과 같은 명령어는 가능한 한 migration 콜백 안에서는 실행하지 않는다.

- 마이그레이션 중에 일관성을 유지하고 싶다면 ‘transaction’ 블록을 감싸서 실행하면 된다. 그렇지만 ‘pragmas’(foreign-keys 포함)은 트랜잭션 안에서 변경될 수 없다.
  - 마이그레이션 전에 foreign-key를 비활성화
  - 데이터베이스를 다시 사용하기 전에 항상 beforeOpen에서 외래 키를 다시 활성화
  - 트랜잭션 안에서 migration을 수행
  - foreign key 제약조건을 비활성화했다면 마이그레이션 과정에서 제약조건을 어기지 않았는 지 확인

> 위의 일관성을 유지하기 위한 4가지 팁을 사용하는 예시 코드는 다음과 같다.
>

```dart
return MigrationStrategy(
  onUpgrade: (m, from, to) async {
    // disable foreign_keys before migrations
    await customStatement('PRAGMA foreign_keys = OFF');

    await transaction(() async {
      // put your migration logic here
    });

    // Assert that the schema is valid after migrations
    if (kDebugMode) {
      final wrongForeignKeys =
          await customSelect('PRAGMA foreign_key_check').get();
      assert(wrongForeignKeys.isEmpty,
          '${wrongForeignKeys.map((e) => e.data)}');
    }
  },
  beforeOpen: (details) async {
    await customStatement('PRAGMA foreign_keys = ON');
    // ....
  },
);
```

‘PRAGMA foreign_keys = OFF’란?

> SQLite에서 'PRAGMA foreign_keys = OFF' 명령어는 외래키 제약조건을 비활성화하는데 사용됩니다. 이 명령은 현재 데이터베이스 세션에서 실행되는 모든 쿼리에 대해 외래키 제약 조건이 적용되지 않도록 합니다.
>
>
> 즉, 이 명령을 실행하면 다른 테이블의 데이터를 참조하는 외래키 제약 조건을 무시하고 데이터베이스에 삽입, 수정 또는 삭제 작업을 수행할 수 있습니다. 이는 데이터베이스를 일시적으로 변경하거나 유지보수 작업을 수행할 때 유용할 수 있습니다.
>
> 하지만, 외래키 제약조건을 무시하는 것은 데이터 일관성을 해칠 수 있으므로, 이 명령어는 주의해서 사용해야 합니다. 외래키 제약 조건을 다시 활성화하려면 'PRAGMA foreign_keys = ON' 명령어를 사용하면 됩니다.
>

### Complex migrations

sqlite는 컬럼을 추가하거나, 전체 테이블을 제거하는 등의 간단한 api를 제공하지만, 이보다 더 복잡한 마이그레이션을 수행하기 위해서는 트랜잭션, foreign key 제약조건, index, view 등 실수할 여지가 많다.
Drift 3.4는 ‘TableMigration’ API를 제공하기 때문에 복잡한 마이그레이션 처리를 위한 과정을 자동화해준다.

TableMigration의 과정은 다음과 같다.

1. ‘TableMigration’은 마이그레이션을 시작하기 위해 현재 스키마의 새로운 인스턴스를 생성한다.
2. 이전 테이블의 Row를 새로 생성한 인스턴스에 복사한다.
3. 하지만 TableMigration은 단순한 컬럼 추가가 아닌 복잡한 과정을 거치기 때문에 기존 테이블과 컬럼의 이름이 다를 수도 있기 때문에 이 경우에는 ‘columnTransformer’를 사용해서 변환을 할 수 있다.
4. ‘columnTransformer’는 이전 테이블에서 열을 복사하는 데 사용된다.

    ```dart
    columnTransformer: {
      todos.category: todos.category.cast<int>(),
    }
    ```


내부적으로 drift는 “INSERT INTO SELECT” 명령을 사용한다.

위의 예의 경우 아마도 아래의 명령어를 사용했을 것이다.

```dart
INSERT INTO temporary_todos_copy SELECT id, title, content, CAST(category AS INT) FROM todos
```

마이그레이션을 진행하는 경우에는 테스트를 진행해서 마이그레이션이 제대로 적용됐는지 확인하는 것이 중요하다.

### Changing the type of a column

```dart
class Todos extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text().withLength(min: 6, max: 10)();
  TextColumn get content => text().named('body')();
-  IntColumn get category => text()();
+  IntColumn get category => integer().nullable()();
}
```

위의 코드를 예시로 특정 컬럼의 타입을 변경하는 방법을 코드를 통해 알아보자. Todos 테이블의 text 타입이었던 category 컬럼을 interger(nullable)로 변경하려고 한다.

마이그레이션 코드는 다음과 같다 .

```dart
return MigrationStrategy(
  onUpgrade: (m, old, to) async {
    if (old <= yourOldVersion) {
      await m.alterTable(
        TableMigration(todos, columnTransformer: {
          todos.category: todos.category.cast<int>(),
        }),
      );
    }
  },
);
```

### During development

개발 단계에서 스키마를 변경할 필요없이 app을 재설치하면 마이그레이션을 할 필요없다.
