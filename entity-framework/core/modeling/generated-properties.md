---
title: 產生的值-EF Core
description: 如何在使用 Entity Framework Core 時設定屬性的值產生
author: AndriySvyryd
ms.author: ansvyryd
ms.date: 11/06/2019
uid: core/modeling/generated-properties
ms.openlocfilehash: 9c616e157ff1bdb9700f436a7ae2788330fe5d45
ms.sourcegitcommit: cc0ff36e46e9ed3527638f7208000e8521faef2e
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/06/2020
ms.locfileid: "78416343"
---
# <a name="generated-values"></a>產生的值

## <a name="value-generation-patterns"></a>值產生模式

有三個可用於屬性的實值產生模式：

* 未產生值
* Add 上產生的值
* 新增或更新時產生的值

### <a name="no-value-generation"></a>未產生值

[無值產生] 表示您一律會提供有效的值以儲存至資料庫。 這個有效值必須先指派給新的實體，才會加入至內容中。

### <a name="value-generated-on-add"></a>Add 上產生的值

[新增] 上產生的值表示會為新實體產生值。

視所使用的資料庫提供者而定，值可能是由 EF 或資料庫中的用戶端產生。 如果值是由資料庫產生，則當您將實體加入內容時，EF 可能會指派暫存值。 此暫存值接著會在 `SaveChanges()`期間由資料庫產生的值取代。

如果您將實體加入至具有指派給屬性之值的內容，則 EF 會嘗試插入該值，而不是產生新的值。 如果未指派 CLR 預設值（`string`的`null`、`int`的 `0`、`Guid.Empty` 等等），則會將屬性視為具有指派的值。`Guid` 如需詳細資訊，請參閱[產生的屬性的明確值](../saving/explicit-values-generated-properties.md)。

> [!WARNING]
> 為新增的實體產生值的方式，將取決於所使用的資料庫提供者。 資料庫提供者可以自動設定某些屬性類型的值產生，但其他則可能需要您手動設定產生值的方式。
>
> 例如，使用 SQL Server 時，將會自動產生 `GUID` 屬性（使用 SQL Server 連續 GUID 演算法）的值。 不過，如果您指定在 add 上產生 `DateTime` 屬性，則必須設定要產生值的方式。 執行此動作的其中一種方式是設定 `GETDATE()`的預設值，請參閱[預設值](relational/default-values.md)。

### <a name="value-generated-on-add-or-update"></a>新增或更新時產生的值

[新增] 或 [更新] 上產生的值表示每次儲存記錄（插入或更新）時，都會產生新的值。

如同 `value generated on add`，如果您在新加入的實體實例上指定屬性的值，就會插入該值，而不是要產生的值。 您也可以在更新時設定明確的值。 如需詳細資訊，請參閱[產生的屬性的明確值](../saving/explicit-values-generated-properties.md)。

> [!WARNING]
> 為新增和更新的實體產生值的方式，將取決於所使用的資料庫提供者。 資料庫提供者可以自動設定某些屬性類型的值產生，而有些則需要您手動設定產生值的方式。
>
> 例如，使用 SQL Server 時，會以 `rowversion` 資料類型來設定在新增或更新時所產生的 `byte[]` 屬性，並標示為並行標記，因此會在資料庫中產生值。 不過，如果您指定在 [新增] 或 [更新] 上產生 `DateTime` 屬性，則必須設定一種方法來產生值。 執行此動作的其中一種方式是設定 `GETDATE()` 的預設值（請參閱[預設值](relational/default-values.md)），以產生新資料列的值。 接著，您可以使用資料庫觸發程式在更新期間（例如下列範例觸發程式）產生值。
>
> [!code-sql[Main](../../../samples/core/Modeling/FluentAPI/ValueGeneratedOnAddOrUpdate.sql)]

## <a name="value-generated-on-add"></a>Add 上產生的值

依照慣例，如果應用程式未提供值，則 short、int、long 或 Guid 類型的非複合主鍵會設定為已為插入的實體產生值。 您的資料庫提供者通常會負責所需的設定;例如，SQL Server 中的數值主鍵會自動設定為識別資料行。

您可以設定任何屬性，為插入的實體產生其值，如下所示：

### <a name="data-annotations"></a>[資料註解](#tab/data-annotations)

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/ValueGeneratedOnAdd.cs?name=ValueGeneratedOnAdd&highlight=5)]

### <a name="fluent-api"></a>[流暢的 API](#tab/fluent-api)

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/ValueGeneratedOnAdd.cs?name=ValueGeneratedOnAdd&highlight=5)]

***

> [!WARNING]
> 這只是讓 EF 知道為新增的實體產生值，並不保證 EF 會設定實際的機制來產生值。 如需詳細資訊，請參閱 add 一節所[產生的值](#value-generated-on-add)。

### <a name="default-values"></a>預設值

在關係資料庫上，可以使用預設值來設定資料行。如果插入的資料列沒有該資料行的值，則會使用預設值。

您可以在屬性上設定預設值：

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/DefaultValue.cs?name=DefaultValue&highlight=5)]

您也可以指定用來計算預設值的 SQL 片段：

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/DefaultValueSql.cs?name=DefaultValueSql&highlight=5)]

指定預設值會將屬性隱含地設定為在 add 上產生的值。

## <a name="value-generated-on-add-or-update"></a>新增或更新時產生的值

### <a name="data-annotations"></a>[資料註解](#tab/data-annotations)

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/ValueGeneratedOnAddOrUpdate.cs?name=ValueGeneratedOnAddOrUpdate&highlight=5)]

### <a name="fluent-api"></a>[流暢的 API](#tab/fluent-api)

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/ValueGeneratedOnAddOrUpdate.cs?name=ValueGeneratedOnAddOrUpdate&highlight=5)]

***

> [!WARNING]
> 這只是讓 EF 知道為新增或更新的實體產生值，並不保證 EF 會設定實際的機制來產生值。 如需詳細資訊，請參閱[add 或 update 一節中產生的值](#value-generated-on-add-or-update)。

### <a name="computed-columns"></a>計算資料行

在某些關係資料庫上，可以將資料行設定為在資料庫中計算其值，通常會有一個運算式參考其他資料行：

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/ComputedColumn.cs?name=ComputedColumn&highlight=5)]

> [!NOTE]
> 在某些情況下，資料行的值會在每次提取時計算（有時稱為「*虛擬*資料行」），而其他則會在資料列的每個更新上進行計算並儲存（有時稱為*儲存*或*保存*的資料行）。 這會因資料庫提供者而異。

## <a name="no-value-generation"></a>未產生值

如果慣例將屬性（property）設定為產生值，通常就需要停用屬性的值產生。 例如，如果您有 int 類型的主鍵，則會隱含地將它設定為在 add 上產生的值;您可以透過下列方式來停用此功能：

### <a name="data-annotations"></a>[資料註解](#tab/data-annotations)

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/ValueGeneratedNever.cs?name=ValueGeneratedNever&highlight=3)]

### <a name="fluent-api"></a>[流暢的 API](#tab/fluent-api)

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/ValueGeneratedNever.cs?name=ValueGeneratedNever&highlight=5)]

***
