# Spring Validation

## Map<String, String> errors
> Model에 직접 넣어서 뷰 템플릿으로 전달하는 방식

> 타입 에러 발생 시 mvc 호출 전에 400 에러 발생하는 문제가 있음 (뷰 템플릿으로 에러 메시지 전달 불가능)
```
@PostMapping("/add")
    public String addItem(@ModelAttribute Item item, RedirectAttributes redirectAttributes, Model model) {

        Map<String, String> errors = new HashMap<>();

        if (!StringUtils.hasText(item.getItemName())) {
            errors.put("itemName", "상품 이름은 필수입니다.");
        }

        if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
            errors.put("price", "가격은 1,000 ~ 1,000,000 까지 허용합니다.");
        }

        if (item.getQuantity() == null || item.getQuantity() >= 10000) {
            errors.put("quantity", "수량은 최대 9,999까지 허용합니다.");
        }

        if (item.getPrice() != null && item.getQuantity() != null) {
            int resultPrice = item.getPrice() * item.getQuantity();
            if (resultPrice < 10000) {
                errors.put("globalError", "가격 * 수량의 합은  10,000 이상이어야 합니다. 현재 값 = " + resultPrice);
            }
        }

        if (!errors.isEmpty()) {
            model.addAttribute("errors", errors);
            return "validation/v1/addForm";
        }

        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/validation/v1/items/{itemId}";
    }
```

## bindingResult 1
> 스프링이 제공하는 오류 검증 보관 객체

> 타입 에러 발생 시 bindingResult에 오류 객체 (특정 필드에서 발생하기 때문에 필드 에러)를 저장하고 mvc 컨트롤러 정상 호출

> Model에 넣지 않아도 뷰 템플릿에서 bindingResult 접근 지원해줌

> 에러 발생 시 사용자가 입력했던 내용이 전부 사라지는 문제가 있음
```
public String addItemV2(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
  '''
  bindingResult.addError(new FieldError("item", "itemName", "상품 이름은 필수입니다."));
  bindingResult.addError(new ObjectError("item", "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));
  '''
  if (bindingResult.hasErrors()) {
            log.info("errors={}", bindingResult);
            return "validation/v2/addForm";
        }
  '''
```
* 필드 에러 : new FieldError(object, field, default message)
* 글로벌 에러 : new ObjectError(object, default message) -> 특정 필드에서 발생한 오류가 아니므로 필드 인자 필요 없음
* bindingResult.addError()로 에러 객체 저장
* 검증 대상 객체(target) 바로 뒤에 선언되어야 함

## bindingResult 2
> 오류 발생 후 입력한 내용을 화면에 유지하기 위해서는 입력한 내용을 저장할 필요가 있음 -> rejectValue
```
    bindingResult.addError(new FieldError("item", "itemName", item.getItemName(), false, null, null, "상품 이름은 필수입니다."));
    bindingResult.addError(new ObjectError("item", null, null, "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));
```
* new FieldError(object, field, rejectValud : 입력했지만 거절된 값, bindingType : 바인딩 성공 여부 : 타입 에러 체크용도, 메시지 코드, 메시지 인자, default message)
* new ObjectError(object, 메시지 코드, 메시지 인자, default message) 

## ⭐️ bindingResult 3
> bindingResult는 오류 발생 시 이미 오류 발생 객체를 알고 있으므로 만들어줄 필요 없음 -> rejectValue(), reject() 함수 사용
```
    bindingResult.rejectValue("itemName", "required", null, null);
    bindingResult.rejectValue("price", "range", new Object[]{1000, 1000000}, null);
    bindingResult.rejectValue("quantity", "max", new Object[]{9999}, null);
    bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
```
* rejectValue(field, errorCode(메시지 리졸버에서 사용됨), 메시지 인자, default message)
* errorCode는 errors.properties에서 맨 처음 단어와 일치해야 메시지 리졸버가 찾을 수 있음
* rejectValue, reject 함수를 사용하면 내부적으로 MessageCodesResolver를 사용하며 이때 errorCode 기준으로 오류 메시지 객체를 생성

## ⭐️ errors.properties
> 구체적인 메시지 (errorcode.object.field)를 먼저 만들고 범용적인 메시지(errorcode)를 가장 나중에 만든다.
```
#==ObjectError==
#Level1
totalPriceMin.item=Level1 상품의 가격 * 수량의 합은 {0}원 이상이어야 합니다. 현재 값 = {1}

#Level2 - 생략
totalPriceMin=Level2 전체 가격은 {0}원 이상이어야 합니다. 현재 값 = {1}

#==FieldError==
#Level1
required.item.itemName=Level1 상품 이름은 필수입니다.
range.item.price=Level1 가격은 {0} ~ {1} 까지 허용합니다.
max.item.quantity=Level1 수량은 최대 {0} 까지 허용합니다.

#Level2 - 생략

#Level3
required.java.lang.String = Level3 필수 문자입니다.
required.java.lang.Integer = Level3 필수 숫자입니다.
min.java.lang.String = Level3 {0} 이상의 문자를 입력해주세요.
min.java.lang.Integer = Level3 {0} 이상의 숫자를 입력해주세요.
range.java.lang.String = Level3 {0} ~ {1} 까지의 문자를 입력해주세요.
range.java.lang.Integer = Level3 {0} ~ {1} 까지의 숫자를 입력해주세요.
max.java.lang.String = Level3 {0} 까지의 문자를 허용합니다.
max.java.lang.Integer = Level3 {0} 까지의 숫자를 허용합니다.

#Level4
required = Level4 필수 값 입니다.
min= Level4 {0} 이상이어야 합니다.
range= Level4 {0} ~ {1} 범위를 허용합니다.
max= Level4 {0} 까지 허용합니다.

#type error (스프링이 직접 검증 오류에 추가한 경우)
typeMismatch.java.lang.Integer=숫자를 입력해주세요.
typeMismatch=타입 오류입니다.
```
* 개발자가 직접 설정한 오류 코드 -> rejectValue()를 직접 호출
* 스프링이 직접 검증 오류에 추가한 경우 -> 주로 타입 에러

## Validator 분리 1
> 검증 로직이 길어지므로 따로 클래스로 분리

> 스프링에서 검증을 위해 사용되는 인터페이스인 Validator 사용
### Validator 구현체
```
@Component
public class ItemValidator implements Validator {
    
    @Override
    public boolean supports(Class<?> clazz) {
        return Item.class.isAssignableFrom(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        Item item = (Item) target;

        if (!StringUtils.hasText(item.getItemName())) {
            errors.rejectValue("itemName", "required", null, null);
        }

        if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
            errors.rejectValue("price", "range", new Object[]{1000, 1000000}, null);
        }

        if (item.getQuantity() == null || item.getQuantity() >= 10000) {
            errors.rejectValue("quantity", "max", new Object[]{9999}, null);
        }

        if (item.getPrice() != null && item.getQuantity() != null) {
            int resultPrice = item.getPrice() * item.getQuantity();
            if (resultPrice < 10000) {
                errors.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
            }
        }
    }
}
```
### 수정된 컨트롤러 (검증 로직 분리)
```
    private final ItemValidator itemValidator;
    
    @PostMapping("/add")
    public String addItemV5(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {

        itemValidator.validate(item, bindingResult);    //검증 로직 분리

        if (bindingResult.hasErrors()) {
            log.info("errors={}", bindingResult);
            return "validation/v2/addForm";
        }

        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/validation/v2/items/{itemId}";
    }
```
* supports() : 해당 검증기를 지원하는 여부 확인
* validate(object, errors) : 검증 대상 객체와 bindingResult (errors의 자식 클래스)

## ⭐️Bean Validation
> 실제로 사용하는 검증 로직

> 검증 로직을 쉽게 사용 가능
### build.gradle
```
	implementation 'org.springframework.boot:spring-boot-starter-validation'
```
### 에노테이션 형태로 검증 
```
@Data
public class Item {


    private Long id;

    @NotBlank
    private String itemName;

    @NotNull
    @Range(min = 1000, max = 1000000)
    private Integer price;

    @NotNull
    @Max(9999)
    private Integer quantity;

    public Item() {
    }

    public Item(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    }
}
```
### ⭐️수정된 컨트롤러
```
    @PostMapping("/add")
    public String addItemV7(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {

        if (item.getPrice() != null && item.getQuantity() != null) {
            int resultPrice = item.getPrice() * item.getQuantity();
            if (resultPrice < 10000) {
                bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
            }
        }

        if (bindingResult.hasErrors()) {
            log.info("errors={}", bindingResult);
            return "validation/v2/addForm";
        }

        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/validation/v2/items/{itemId}";
    }
```
### ⭐️errors.properties
```
#type error
typeMismatch.java.lang.Integer=숫자를 입력해주세요.!!
typeMismatch=타입 오류입니다.!!

#Bean Validation 추가
NotBlank={0} 공백 싫어요
Range={0}, {2} ~ {1} 허용 좋아요
Max={0}, 최대 {1}
```
* 스프링 부트가 자동으로 Validator 등록
* 검증 시 @Validated, @Valid 중 하나를 검증 객체 선언 앞에 추가, 에러 발생 시 bindingResult에 담긴다.
* 검증 순서는 @ModelAttribute -> 각각의 field 타입 변환 시도 -> 변환 실패 시 typeMismatch로 FieldError 추가 -> 성공한 field만 BeanValidation으로 검증
* 생성 메시지 코드 로직은 이전과 같다. (NotBlank.item.itemName ...... NotBlank  순서의 우선순위)
* errors.properties에서 {0}은 필드명(생략 가능), {1} ~ {n}은 인수
* 필드 오류가 아닌 글로벌 오류는 스프링에서 지원하는 기능이 약하므로 직접 자바 코드로 작성하는 것을 추천

### DTO
> Item 도메인에 대하여, addItem 로직에서 필요한 정보와 editItem 로직에서 필요한 정보가 각각 다르다. (실제로는 정보 간 차이가 더욱 크다.)

> 따라서 Item 도메인과 그것을 add, edit 하는 로직에서 각각 사용하는 DTO (Data Transfer Object, 혹은 폼 데이터 전달)를 따로 분리하는 것이 좋다.

> 상속이나 스프링에서 지원하는 groups는 편리하지만 간단한 경우에만 사용 가능하다. (각 객체 간 정보 차이가 다를 수 있기 때문)
