# Лаб 6 — Category-Partition ба PICT

Оюутны нэр — Я. Төгөлдөр

Оюутны код — B242270806

| Сонголт | Анги ба төлөөлөх утга | Тэмдэглэгээ |
|---|---|---|
| From | Valid=A, Frozen=F, Missing=X | Frozen, Missing [ERROR] |
| To | Valid=B, Missing=Y, SameAsFrom=A эсвэл F | Missing, SameAsFrom [ERROR], SameAsFrom [IF] байж буй эх данс |
| Amount | Normal=20, Zero=0, OverBalance=120, OverLimit=80 | Сүүлийн гурав [ERROR], Zero [SINGLE] ханшийн хувьд, OverLimit [IF] лимитэд зайтай |
| Currency | Same=MNT, Different=FX | FX ханш 2 MNT, хөрвүүлсний дараах дүнгээр ангилна |
| Limit | Open=60 MNT үлдсэн, Exhausted=0 MNT | Exhausted [ERROR] эерэг дүнд, [IF] эх данс байгаа |

Эх дансны үлдэгдэл 100 MNT, FX төлөөлөх дүн нь ижил ангиллын MNT дүнгийн тал. Эх данс байхгүй бол тухайн дансны үлдэгдэл, бодит лимит ба ижил данс гэсэн нөхцөл утгагүй тул amount-ийг Normal эсвэл Zero, лимитийг Open гэж тогтооно. Zero-д валют давтаж шалгах шаардлагагүй, 0 дүн хөрвүүлсэн ч 0 хэвээр. OverLimit нь Open үед 60-аас их, үлдэгдэлд багтсан дүнг төлөөлнө. Exhausted нь энэ ангийг давхардуулах тул OverLimit-тэй хослуулахгүй.

Алдааны дараалал нь NO_FROM → FROZEN → NO_TO → SAME_ACCOUNT → BAD_AMOUNT → INSUFFICIENT → LIMIT. Дансны таних өгөгдөл хүчинтэй болсны дараа мөнгөн дүнг шалгах учраас энэ дарааллыг сонгов.

Бүрэн олонлог 3 × 3 × 4 × 2 × 2 = 144. Байгаа эх данс бүрд нэг to утга тутамд Normal 4, Zero 2, OverBalance 4, OverLimit 2 хослол байна. Missing эх дансанд боломжтой хоёр to утга тус бүрд Normal 2, Zero 1 хослол бий. Иймээс 144 → 78 буюу 2 × 3 × 12 + 2 × 3.

| Кейс | From | To | Amount | Currency | Limit | Oracle |
|---|---|---|---|---|---|---|
| 1 | Valid | Valid | Normal | Same | Open | 201 OK |
| 2 | Missing | Valid | Normal | Same | Open | 200 ERROR_NO_FROM |
| 3 | Frozen | Valid | Normal | Same | Open | 200 ERROR_FROZEN |
| 4 | Valid | Missing | Normal | Same | Open | 200 ERROR_NO_TO |
| 5 | Valid | SameAsFrom | Normal | Same | Open | 200 ERROR_SAME_ACCOUNT |
| 6 | Valid | Valid | Zero | Same | Open | 200 ERROR_BAD_AMOUNT |
| 7 | Valid | Valid | OverBalance | Same | Open | 200 ERROR_INSUFFICIENT |
| 8 | Valid | Valid | OverLimit | Same | Open | 200 ERROR_LIMIT |
| 9 | Frozen | Missing | Normal | Same | Open | 200 ERROR_FROZEN |
| 10 | Valid | Valid | Normal | Different | Open | 201 OK |
| 11 | Valid | Valid | Normal | Different | Exhausted | 200 ERROR_LIMIT |
| 12 | Valid | Valid | OverBalance | Different | Exhausted | 200 ERROR_INSUFFICIENT |

Браузерын бүрэн олонлог 3 × 3 × 2 × 2 × 2 × 2 = 144. PICT greedy сонголтоор нэг мөрд тухайн үед хамрагдаагүй олон хосыг багтаахыг зорьдог. Эхний сонголтууд дараагийн сонголтыг хязгаарладаг тул гараар олсон 9 мөрийн шийдэлтэй ижил тоог баталгаажуулдаггүй.

`pict browser.model`, `pict browser.model /o:3`, `pict transfer-nc.model`, `pict transfer.model`, `pict transfer-neg.model` гэсэн командуудаар гаралт авна. Гаралтын толгойн мөрийг тестийн тоонд оруулахгүй. `~` утга олон алдааны харилцан халхлалтыг арилгана. Zero ба валютын давхардлыг арилгах дүрэм нь алдааны тоог хязгаарлахаас тусдаа тул negative модельд үлдэнэ.

PICT-ийг `ab76c25` commit-оос LLVM MinGW clang++ 23.1.6 ашиглан Windows x64 дээр бүтээсэн. C++17 горимд `api/*.cpp` ба `cli/*.cpp` файлуудыг `-static -municode -DUNICODE -DWIN32_LEAN_AND_MEAN -Iapi` тохиргоотой хөрвүүлж холбосон. [Build ба CLI хэрэглээний гаралт](results/pict-build.txt)

| Гаралт | Мөр | Happy path | Алдааны төлөвтэй баганын дээд тоо |
|---|---:|---:|---:|
| [Browser 2-way](results/browser-2way.txt) | 10 | — | — |
| [Browser 3-way](results/browser-3way.txt) | 20 | — | — |
| [Transfer, хязгаарлалтгүй](results/transfer-nc.txt) | 14 | 1 | 4 |
| [Transfer, хязгаарлалттай](results/transfer.txt) | 14 | 0 | 3 |
| [Transfer, negative](results/transfer-neg.txt) | 17 | 2 | 1 |

Browser 2-way гараар хийсэн 9 мөрөөс нэгээр олон, 3-way бүтэн 144 хослолын 20 мөрийг сонгосон. Transfer-ийн хязгаарлалтууд мөрийн тоог хэвээр үлдээж, нэг мөрийн хамгийн их алдааны төлөвийг 4-өөс 3 болгосон. Negative хувилбар нэг алдаатай мөрүүдийг тусад нь гаргасан тул мөрийн тоо өссөн. Хязгаарлалттай pairwise модельд happy path бүх нөхцөл нэг мөрд таараагүй тул тусдаа сонгож шалгах нь зүйтэй.

## PICT гаралтаас авсан кейс

Мөрийн дугаар толгойг хассан. Setup нь кейс бүрд тусдаа. FX ханш 2 MNT.

| Модель ба мөр | Setup | POST /transfer-ийн JSON | Oracle |
|---|---|---|---|
| transfer-nc, 12 | A идэвхтэй, 100 MNT үлдэгдэлтэй, лимитэд 60 MNT зайтай. B байна. | `{"from":"A","to":"B","amount":10,"currency":"FX"}` | HTTP 201, `result=OK` |
| transfer, 3 | F царцаалттай, 100 MNT үлдэгдэлтэй, лимитэд 60 MNT зайтай. Y байхгүй. | `{"from":"F","to":"Y","amount":60,"currency":"FX"}` | HTTP 200, `result=ERROR_FROZEN` нь тогтоосон эрэмбээр эхэлж ирнэ. |
| transfer-neg, 6 | A идэвхтэй, 100 MNT үлдэгдэлтэй, лимитэд 60 MNT зайтай. B байна. | `{"from":"A","to":"B","amount":0,"currency":"MNT"}` | HTTP 200, `result=ERROR_BAD_AMOUNT` |

## Дүгнэлт

Гар аргаар ангилахдаа дүнгийн ангиллыг хөрвүүлсний дараах утгаар тодорхойлсон. Эх данс байхгүй үед бодит үлдэгдлийг мэдэх боломжгүй учраас зарим хослолыг хассан. Өдрийн лимит дүүрсэн төлөв нь хүсэлтийн биед харагддаггүй ч үр дүнд нөлөөлнө. Давхар алдааны oracle-ийг алдааны давуу эрэмбээр тогтоосон. Гар хүснэгт энэ эрэмбийг зориуд сонгож шалгах боломж олгодог. PICT нь хүчинтэй утгуудын хосын хамралтыг өгдөг. Negative модель тусгаарласан алдааны ангийг шалгахад тохирно. Ажлын хамгийн их анхаарал шаардсан хэсэг нь боломжгүй хослолыг хасахдаа утгатай давхар алдааг хадгалах байв.
