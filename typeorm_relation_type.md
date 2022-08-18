# TypeORM relations
- TypeORM은 JavaScript/TypeScript에서 사용할 수 있는 ORM이며 쿼리를 안전하게 할 수 있게 도와주기도 하지만 모든 경우가 해당되지는 않는다.
- 이번 글에서 적고자 하는 `relations`에 대해서는 일반적으로 `string[]`으로 처리하고 있다.
- ORM을 사용하여 개발을 함에 있어 Type Safety한 환경에서 개발을 원하는데 위와 같은 타입으로는 컴파일시에 오류를 잡지 못하고 런타임시에 오류를 발생 시킬 수 있다.
- relations에 사용 할 수 있는 column들은 이미 entity 파일에 정의되어 있는데 컴파일시에 오류를 방지 할 수 없다는게 이해가 되지 않아 relations에 사용 할 수 있는 column만을 입력 할 수 있는 Type을 추가해보기로 하였다.

## Entity
```ts
@Entity('order')
class OrderEntity {
  @PrimaryGeneratedColumn({ type: 'bigint' })
  id: number;

  @OneToMany(type => OrderItemEntity, (item) => item.order)
  @Column()
  items: OrderItemEntity[];

  @Column(type: 'json')
  payJson: PayJson;
}

@Entity('order_item')
class OrderItemEntity {
  @PrimaryGeneratedColumn({ type: 'bigint' })
  id: number;

  @ManyToOne(type => OrderEntity, (order) => order.items)
  order: OrderEntity;

  @Column({ type: 'tinyint', nullable: false })
  amount: number;

  @Column({ type: 'mediumint', nullable: false })
  price: number;
}
```
- 위 Entity 중 @OneToMany, @ManyToOne 같은 데코레이터가 선언된 column에 대해서만 relations 값으로 사용하고 싶다<br>
ex) relations: ('items' | ... | ...)[] = [];

***

### step 1. relations에 사용되는 column type은 보통 class이다. class를 사용하는 key를 filter 해보자
```ts
type ExcludeColumn = string | number | boolean | '...';

type FilterConditionally<Source, Condition> = Omit<
  Source,
  { [K in keyof Source]: Source[K] extends Condition ? K : never }[keyof Source]
>;

type Relations<T> = keyof FilterConditionally<T, ExcludeColumn>;

const relations: Relations<Order>[] = []; // ('items' | 'payJson')[]
```
- json type의 column이 있으면 의도대로 필터되지 않는다

***

### step 2. relations을 나타내는 타입을 wrapping하자
```ts
type Relation<T> = T;

type IncludeColumn<T> = Relation<T>;

type FilterConditionally<Source, Condition> = Pick<
  Source,
  { [K in keyof Source]: Source[K] extends Relation<Condition> ? K : never }[keyof Source]
>;
```
- 단순히 wrapping하는 것으로는 다른 타입으로 분류되지 않는다

***

### step 3. 가상의 타입을 만들자
```ts
type Brand<key extends string, Value> = Value & { __brand?: Key };

type Relation<T> = Brand<'Relation', T>;

type IncludeColum = Relation<unknown>;

class OrderEntity {
  @OneToMany(type => OrderItemEntity, (item) => item.order)
  @Column()
  items: Relation<OrderItemEntity[]>;
}

const relations: Relations<Order>[] = []; // ('items')[]
```

***

```ts
// typeorm.d.ts
export declare module 'typeorm' {
  declare interface FindManyOptions<Entity> {
    relations?: Relations<Entity>[]; // AS IS - relations?: string[];
  }
}
```
이렇게 기존의 relations를 string[] 대신 Relations<Entity>[]를 사용하게 하면 완벽하지는 않지만 의도하지 않은 오타나 잘못된 컬럼값이 들어가지 않을 수 있게 되었다.