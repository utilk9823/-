**KEY WORD**
 - 物件導向
  - 父繼承
 
我在看`llama-index-core/llama_index/core/retrievers/transform_retriever.py`的時候，發現了以下的程式碼。

```
from typing import List, Optional

from llama_index.core.base.base_retriever import BaseRetriever
from llama_index.core.callbacks.base import CallbackManager
from llama_index.core.indices.query.query_transform.base import BaseQueryTransform
from llama_index.core.prompts.mixin import PromptMixinType
from llama_index.core.schema import NodeWithScore, QueryBundle


class TransformRetriever(BaseRetriever):
    """Transform Retriever.

    Takes in an existing retriever and a query transform and runs the query transform
    before running the retriever.

    """

    def __init__(
        self,
        retriever: BaseRetriever,
        query_transform: BaseQueryTransform,
        transform_metadata: Optional[dict] = None,
        callback_manager: Optional[CallbackManager] = None,
        object_map: Optional[dict] = None,
        verbose: bool = False,
    ) -> None:
        self._retriever = retriever
        self._query_transform = query_transform
        self._transform_metadata = transform_metadata
        super().__init__(
            callback_manager=callback_manager, object_map=object_map, verbose=verbose
        )

    def _get_prompt_modules(self) -> PromptMixinType:
        """Get prompt sub-modules."""
        # NOTE: don't include tools for now
        return {"query_transform": self._query_transform}

    def _retrieve(self, query_bundle: QueryBundle) -> List[NodeWithScore]:
        query_bundle = self._query_transform.run(
            query_bundle, metadata=self._transform_metadata
        )
        return self._retriever.retrieve(query_bundle)
```

我沒看懂其中的
```
super().__init__(
            callback_manager=callback_manager, object_map=object_map, verbose=verbose
        )
```

而我詢問**Copilot**得到以下解答

```
這段程式碼 super().__init__(callback_manager=callback_manager, object_map=object_map, verbose=verbose) 是在呼叫 TransformRetriever 類別的父類別的初始化方法 (__init__)。

以下是這段程式碼的詳細解釋：

super()：這個函數會返回父類別的臨時對象，讓你可以呼叫父類別的方法。
__init__：這是父類別的初始化方法。
callback_manager=callback_manager：將 callback_manager 參數傳遞給父類別的 __init__ 方法。
object_map=object_map：將 object_map 參數傳遞給父類別的 __init__ 方法。
verbose=verbose：將 verbose 參數傳遞給父類別的 __init__ 方法。
簡而言之，這行程式碼確保了父類別的初始化邏輯被執行，讓 TransformRetriever 繼承並正確初始化父類別定義的任何屬性或方法。
這是物件導向程式設計中的常見做法，以確保子類別能正確設置並繼承父類別的必要屬性和行為。
```

顯然這是有關Python物件導向的知識，如果以比較基本的範例來看，會同等於以下邏輯。

```
class Animal:
    def __init__(self, name, age):
        self.name = name
        self.age = age

class Dog(Animal):
    def __init__(self, name, age, breed):
        super().__init__(name, age)
        self.breed = breed
```

因此`super().__init__`是TransformRetriever繼承了BaseRetriever父類別的屬性，更進一步的父類別的屬性可以再詳閱`llama_index/llama-index-core/llama_index/core/base
/base_retriever.py`
