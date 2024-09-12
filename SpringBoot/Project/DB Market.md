---
tags:
  - fragment
  - spring
  - "#project"
text: "[[Spring]]"
---

프로젝트를 잘 마친거 같지만, 이 정보들이 온전히 나의 것이라고는 생각하지 않는다.
꼭꼭 씹어먹으면서 나의 것으로 소화해 나가는 과정이 필요하다.

# 1. Entity - DTO

~~~java title:"Entity"
@AllArgsConstructor  
@NoArgsConstructor  
  
@Setter  
@ToString  
@Getter  
@Entity  
@Table  
public class User {  
    @Id   
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long userId;  
  
    @Column(nullable = false)  
    private String userName;  
  
    @Column(nullable = false)  
    private String password;
}    
~~~

~~~java title:"DTO"
@Getter  
@AllArgsConstructor  
public class UserDto {  
    private String username;  
    private String password;
~~~

[[2. SpringBoot 주요 개념#DAO, DTO, VO|DTO]] : 계층간 데이터 교환을 위한 객체
[[8. DataBase Modeling#Entity|Entity]] : 데이터 베이스에 저장되는 객체 

### DTO와 Entity 분리의 장점
사실 이번 프로젝트에 아쉬운 부분은 UserDTO 보다는 User 엔티티를 그냥 불러와서 썼다는 점이다.

엔티티를 가져와서 쓸때 편한점  (대부분 귀찮은 부분들인듯)
- ID 값을 사용할 수 있다.
- 선언할 때 `User user` 가 `UserDto user` 보다 더 직관적인건 사실이다.
- DTO를 쓴다면 레파지토리에서 User를 가져온다음 한번 변환작업을 거쳐야 하는 불편함이 있다.

DTO를 도입해야 할 시점
- 시스템 **규모가 커지고 복잡**해질 때: 엔티티와 뷰 사이의 데이터를 효과적으로 관리하기 위해
- 다양한 클라이언트에서 데이터를 사용할 때: 각 클라이언트에 맞는 데이터 형식으로 변환하기 위해 DTO를 사용하면 편리
  엔티티를 사용하면 모든 데이터가 반환되기 때문에, DTO를 통해서 필요한 부분만 전송할 수 있다.
- 데이터 **보안**이 중요할 때: 민감한 정보를 노출시키지 않기 위해

DTO 분리의 장점
- 엔티티의 구조가 변해도 클라이언트에 직접적으로 영향을 미치지 않는다. (영향 최소화)
- 엔티티 보호가능 (DTO에서만 Setter 사용시)
- 보낼 데이터를 선별가능 : 엔티티에서 필요한 데이터만 DTO로 보내면 보안, 최적화 둘다 잡을 수 있다


# 2. 의존관계 주입

~~~java
@RequiredArgsConstructor  
@Controller  
public class UserController {  
    private final UserService userService;  
    private final UserFilter userFilter;
	.
	.
	.
~~~

`{java}@RequiredArgsConstructor` : final이 붙은 필수 필드들을 자동으로 [[6. 의존관계 자동주입#final 키워드|생성자]]를 만들어주면서 의존주입도 동시에 일어난다.
-> `{java}@Autowired` 가 필요없다!

지금은 UserService, UserFilter 클래스를 자동으로 의존관계 주입해줬다.
개인적으로 이방식이 가장 간결한것 같다.

# 3. Repository

~~~java title:"Repository"
public interface UserRepository extends JpaRepository<User, Long> {  
    // 로그인 로직  
    User findByUserNameAndPassword(String username, String password);  
    User findByUserName(String userName);  
    User findByUserId(Long userId);  
}
~~~
저장소 역할



# 3. 로그인
~~~java title:"컨트롤러"
@PostMapping("/users/login")  
public String login(@RequestParam("username") String username, 
					@RequestParam("password") String password, 
					HttpServletRequest httpServletRequest,Model model) {  
    userService.isSessionAvailable(model);  
    return userService.login(username, password, httpServletRequest);  
}
~~~

~~~java title:"서비스"
// 로그인 로직  
public String login(String username, String password, HttpServletRequest httpServletRequest) {  
    // 입력한 username-password 를 모두 충족하지 못하면 null    
    User user = userRepository.findByUserNameAndPassword(username, password);  
  
    // 로그인 검증 로직  
    if (user == null) {  
        log.info("로그인 실패 : 존재하지 않는 계정");  
        return "redirect:/users/login?error= not_exist";  
    }    String password1 = Encrypt.md5(password);  
    String password2 = Encrypt.md5(user.getPassword());  
  
    if (!password1.equals(password2)) {  
        log.info("로그인 실패 : 틀린 비밀번호");  
        return "redirect:/users/login?error=wrong_password";  
    }    log.info("로그인 성공, 세션 부여");  
    HttpSession httpSession = httpServletRequest.getSession(true);  
    httpSession.setAttribute("user", user);  
  
    return "redirect:/categories";  
}
~~~

[[3. MVC Pattern|MVC 패턴]] 을 한번에 보여주는 케이스

`{java}@RequestParam("파라미터 이름")` 을 통해 뷰페이지에서 입력한 내용을 매개변수로 가져와 담을 수 있다.

컨트롤러는 매개변수를 서비스로 보내는 단순한 역할만 한다.
[[10. Service, Transaction|서비스]]는 비즈니스 로직을 수행한다.

- 레파지토리에서 아이디와 비밀번호를 가져온다.
- 검증로직을 통해 로그인 성공 여부 결정
- 로그인 성공하면 세션 부여
- URL 반환


# 4. UserFilter
~~~java
@RequiredArgsConstructor  
@Slf4j  
@Component  
public class UserFilter implements Filter {  
  
    private final HttpSession httpSession;  
    User user;  
    // 화이트리스트 링크
    private static final String[] whiteList = {"/","/home", "/users/join","/users/login"};  
  

    // 요청이 올 때 마다 해당 메서드가 호출  
    @Override  
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) 
							    throws IOException, ServletException {  
  
        HttpServletRequest httpRequest = (HttpServletRequest) request;  
        String requestURI = httpRequest.getRequestURI();  
        HttpServletResponse httpResponse = (HttpServletResponse) response;  
    
        try{  
	        // 화이트리스트 URL 아닌경우
	        if(isLoginCheckPath(requestURI)){  
		        // 세션이 없다면 home으로 강제 리다이렉트
	            if(httpSession.getAttribute("user") == null){  
		            httpResponse.sendRedirect("/home");  
	                return;  
                }           
	        }           
	        filterChain.doFilter(request, response);  
	    }       
	    catch (Exception e){throw e;}  
    }  
    
    public User findUserByFilter(Model model, HttpServletRequest request) {  
        user = (User) request.getSession().getAttribute("user");  
            model.addAttribute("myusername", user.getUserName());  
            model.addAttribute("myuserid", user.getUserId());  
        return user;  
    }  
    
    private boolean isLoginCheckPath(String requestURL){  
        return !PatternMatchUtils.simpleMatch(whiteList,requestURL);  
    }
~~~
세션과 관련된 부분은 따로 클래스로 빼도 좋을 듯 - [[0. 오브젝트와 의존관계#리펙토링 관계설정 책임의 분리|관심사의 분리]]


# 5. 회원가입
컨트롤러
~~~java
@GetMapping("/users/join")  
public String joinForm(Model model) {  
    log.info("회원가입 페이지 요청 접수");  
    userService.isSessionAvailable(model);  
    return "users/join";  
}  
  
@PostMapping("/users/join")  
public String CreateUser(UserDto form, @RequestParam("password_check") String password_check, HttpServletRequest httpServletRequest, Model model) {  
    log.info("회원가입 요청 접수");  
    userService.isSessionAvailable(model);  
    return userService.createUser(form, password_check, httpServletRequest);  
}
~~~

서비스
~~~java
public String createUser(UserDto form, String password_check, HttpServletRequest httpServletRequest) {  
    User user = form.toEntity();  
    User name = userRepository.findByUserName(form.getUsername());  
  
    if (user.getUserName().contains(" ") || user.getPassword().contains(" ") 
    || user.getUserName().isEmpty() || user.getPassword().isEmpty()) {  
        return "users/join";  
    }    String password1 = Encrypt.md5(user.getPassword());  
    String password2 = Encrypt.md5(password_check);  
    if (!password1.equals(password2)) {  
        log.info("회원가입 실패 : 비밀번호 불일치");  
        return "users/join";  
    }  
    if (name == null) {  
        userRepository.save(user);  
        log.info("회원가입 성공, 세션 부여");  
        HttpSession httpSession = httpServletRequest.getSession(true);  
        httpSession.setAttribute("user", user);  
        log.info("계정 DB저장 성공");  
        return "redirect:/categories";  
  
    } else {  
        log.info("회원가입 실패 : 이미 존재하는 계정");  
        return "redirect:/users/join?error=already_exists";  
    }
}
~~~

##### 아쉬운점 1
기껏 password 암호화를 해놓고 암호화를 하지 않은 상태로 DB에 넣어버렸다.
비밀번호 같은 경우는 복호화를 아예 하지 않는다는 마인드로 접근하는게 낫다
암호화된 상태로 DB에 넣고 나중에 DB에서 암호화된 상태로 꺼내서 비교하면 됨

##### 아쉬운점 2
암호화의 수준이 낮다.
다음 프로젝트에서는 해시화를 2중으로 한다던지, 한번더 나만의 방식으로 꽈버려도 좋을듯 
**해시솔트** 참고 할 것

##### 아쉬운점 3
다음번에는 `@Validation` - 정규표현식을 사용해서 더 디테일하게 폼을 제한해볼 것


# 6. Category
~~~java
@Table(name="category")  
@Entity  
@Getter  
@Builder  
@ToString  
@NoArgsConstructor  
@AllArgsConstructor  
@EntityListeners(AuditingEntityListener.class)  
public class Category {  
  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long categoryId;  
  
    @OneToMany(fetch = FetchType.LAZY, mappedBy = "category")  
    private List<Item> items;  
  
    private String name;  
  
    @CreatedDate  
    private LocalDateTime createdAt;  
  
    @LastModifiedDate  
    private LocalDateTime updatedAt;  
}
~~~

내가 작성한 코드는 아니지만, 인사이트를 하나 얻음
[[6. ENUM|ENUM]] vs DB화
 
- ENUM 장점 : 메모리(리소스) 이점, 잘못된 값을 허용하지 않음
- ENUM 단점 (DB화의 장점) : 신규 카테고리를 적용할 때 서버를 재시작할 필요가 없다. 


# 7. User Delete
컨트롤러
~~~java
@DeleteMapping("/users/delete")  
public String deleteUser(Model model, HttpServletRequest httpServletRequest) {  
    User user = userFilter.findUserByFilter(model, httpServletRequest);  
    userService.deleteUser(user.getUserId());  
  
    return "redirect:/users/logout";  
}
~~~

서비스
~~~java
public void deleteUser(Long userId) {  
    User user = userRepository.findByUserId(userId);  
    // 작성한 판매글 List 삭제  
    List<Item> items = itemDetailRepository.findAllByUserId(user.getUserId());  
    for (Item item : items) {  
        log.info("\n회원이 작성한 게시글 : {} 삭제 \n", item.getName());  
        itemDetailRepository.delete(item);  
    }  
    // 탈퇴한 회원이 작성한 코멘트 삭제  
    List<Comment> comments = commentRepository.findAllByUser_UserId(user.getUserId());  
    for (Comment comment : comments) {  
        log.info("\n회원이 작성한 코멘트 : {} 삭제 \n", comment.getContent());  
        commentRepository.delete(comment);  
    }  
    // 회원에게 작성된 코멘트 삭제  
    List<Comment> comments1 = commentRepository.findAllByReviewerName(user.getUserName());  
    for (Comment comment : comments1) {  
        log.info("\n회원에게 작성된 코멘트 : {} 삭제 \n", comment.getContent());  
        commentRepository.delete(comment);  
    }  
    userRepository.delete(user);  
    log.info("\n회원 삭제 : {} 삭제 \n", user.getUserName());  
}
~~~

##### 아쉬운점 
DB의 수정, 삭제의 경우는 아주 조심해야 한다.
오류, 반복된 요청이 발생할 경우 로직이 안정적으로 수행되어야 하는데
`@Transactional` 이 필요할 듯 하다.

##### 의외인점
`@Cascade` 를 사용하려다가 잘 안됐었다.
근데 실무에서도 이런식으로 직접 찾아서 삭제하는 방식을 권장하는 듯 하다.
DB의 수정 삭제가 그만큼 위험한 것이다.

# 8. UserUpdate
컨트롤러
~~~java
@GetMapping("/users/edit")  
public String editUser(Model model, HttpServletRequest httpServletRequest) {  
    User user = userFilter.findUserByFilter(model,httpServletRequest);  
    model.addAttribute("user", user);  
  
    return "users/edit";  
}  
  
@PostMapping("/users/editName")  
public String editUserName(Model model, @RequestParam("username") String newUserName,HttpServletRequest request) {  
    User user = userFilter.findUserByFilter(model, request);  
    return userService.editUserName(user,newUserName,request);  
}  
  
@PostMapping("/users/editPassword")  
public String editUserPassword(Model model,@RequestParam("password") String newPassword, @RequestParam("password_check") String newPasswordCheck, HttpServletRequest request) {  
    User user = userFilter.findUserByFilter(model, request);  
    return userService.editUserPassword(user, newPassword, newPasswordCheck);  
}
~~~


서비스
~~~java
public String editUserName(User user, String newUserName, HttpServletRequest httpServletRequest) {  
    // 기존 아이디를 재 입력했을 경우  
    if (user.getUserName().equals(newUserName)) {  
        log.info("아이디 수정 실패 : 기존 아이디를 재입력함");  
        return "redirect:/users/edit?notNew=error";  
    }    // 이미 존재하는 아이디일 경우  
    User searchUser = userRepository.findByUserName(newUserName);  
    if (searchUser != null) {  
        log.info("아이디 수정 실패 : 이미 존재하는 계정");  
        return "redirect:/users/edit?alreadyExists=error";  
    }    // 성공할 경우  
    User target = userRepository.findByUserName(user.getUserName());  
    target.setUserName(newUserName);  
    userRepository.save(target);  
    log.info("\n회원 아이디 변경 {} -> {} ", user.getUserName(), target.getUserName());  
    HttpSession httpSession = httpServletRequest.getSession(true);  
    httpSession.setAttribute("user", target);  
    return "redirect:/users/edit?Success=Success";  
}
~~~

~~~java
public String editUserPassword(User user, String newPassword, String newPasswordCheck) {  
    String oldPassword = Encrypt.md5(user.getPassword());  
    String nuPassword = Encrypt.md5(newPassword);  
    String nuPasswordCheck = Encrypt.md5(newPasswordCheck);  
  
    // 비밀번호 불일치  
    if (!nuPassword.equals(nuPasswordCheck)) {  
        log.info("비밀번호 수정 실패 : 비밀번호가 일치하지 않음");  
        return "redirect:/users/edit?PasswordNotCorrect=error";  
    }    // 기존 비밀번호 재 입력  
    if (nuPassword.equals(oldPassword)) {  
        log.info("비밀번호 수정 실패 : 기존 비밀번호와 동일함");  
        return "redirect:/users/edit?PasswordNotNew=error";  
    }    // 성공  
    User target = userRepository.findByUserName(user.getUserName());  
    target.setPassword(newPassword);  
    userRepository.save(target);  
    log.info("비밀번호 수정 성공");  
    return "redirect:/users/edit?PasswordUpdateSuccess=Success";  
}
~~~


# 9. 보이지 말아야 할것
안보여주면 아무 오류가 생기지 않는것을 굳이 보여주고 오류 처리하는 것은 매우 비효율적이라고 생각함

~~~java
// 헤더에 마이페이지, 로그아웃 안보이게 하는 로직  
public void isSessionAvailable(Model model) {  
    model.addAttribute("isSessionAvailable", "false");  
}
~~~

[[Front]]
~~~HTML hl:2
<div class="container mx-auto px-4 flex justify-end items-left" 
	 th:if="${isSessionAvailable != 'false'}">  
  
    <div class="mx-3 my-2">  
		<button type="button" id="mypage-btn">  
		    <a th:href="@{/users/mypage}"  
		       class="생략">  
		        <i data-feather="user" class="mr-2"></i> 마이페이지  
		    </a>  
		</button>  
    </div>  
  
    <div class="mx-3 my-2">  
		<button type="button" id="logout-btn">  
		    <a th:href="@{/users/logout}"  
		       class="생략">  
			    <i data-feather="log-out" class="mr-2"></i> 로그아웃 
			</a>  
		</button>  
	</div> 
</div>
~~~

# 10. 홈화면, 로그아웃

홈화면
~~~java
// 첫 홈 화면  
@GetMapping("/home")  
public String welcomeHome(Model model, HttpServletRequest request) {  
    HttpSession session = request.getSession();  
    if (session.getAttribute("user") != null) {  
        log.info("로그인 세션 존재 - categories로 이동");  
        return "redirect:/categories";  
    } else {  
       userService.isSessionAvailable(model);  
        return "home";  
    }
}
~~~

로그아웃 - 서비스
~~~java
public String logout(HttpServletRequest httpServletRequest, Model model) {  
    if (httpServletRequest.getSession().getAttribute("user") == null) {  
        log.info("이미 세션이 없으므로 home으로 리다이렉트");  
        return "home";  
    } else {  
        log.info("세션제거, home으로 리다이렉트");  
        HttpSession session = httpServletRequest.getSession(false);  
        session.invalidate();  
        return "home";  
    }
}
~~~

[[1. Servlet#세션 관리 기능|로그인 세션]]을 조회해서 적절한 페이지로 넘겨준다.
대부분의 페이지가 세션을 필요로 해서, 세션이 없는 상태에서 페이지를 왔다갔다 하면 의도치 않은 오류가 생긴다.


# 11. 아이템 그리드 형식으로 배열
~~~HTML hl:2,23
<div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-3 gap-4">  
    <div th:each="item : ${items}" class="relative group">  
        <a th:href="@{/items/{id}(id=${item.itemId})}" class="block">  
            <img th:src="${item.imagePath}" alt="image" 
		        class="w-full h-full object-cover rounded-lg transition duration-300 transform 
				    group-hover:scale-105">  
        </a>  
  
        <div class="absolute bottom-0 left-0 right-0 p-4 bg-white bg-opacity-80 
	        rounded-b-lg transition duration-300 opacity-0 group-hover:opacity-100">  
            <a th:href="@{/items/{id}(id=${item.itemId})}" class="block">  
                <h2 class="text-lg font-medium text-center">  
                    <span th:text="${item.name}"></span>  
                </h2>  
                <p class="text-gray-700 text-center">  
                    <span th:text="${item.category}"></span>  
                </p>  
                <p class="text-gray-700 text-center">  
                    가격: <span th:text="${item.price}"></span>원  
                </p>  
            </a>  
        </div>  
    </div>  
</div>
~~~

# 12. Header, Footer
[[1.  MVC#뷰 템플릿 레이아웃|뷰 템플릿 헤더, 푸터]] [[15. Fragment|Fragment 설정]] 
~~~HTML
<html xmlns:th="http://www.thymeleaf.org"  
      th:replace="~{fragments/layout :: layout(~{::title}, ~{::main})}">
~~~






# 다음에 적용해 보면 좋을 것들
- API 
- AOP
- REST API - JSON 활용
- 해시솔트 - 암호화 제대로 구현
- @Transactional
- `@RequestBody` 로 JSON을 객체로 받아서 `@ResponseBody` 로 JSON 반환 -> [[8. Request#HTTP 요청 메시지 - JSON|참고]]
- DB [[6. JOIN|JOIN]] 같은 복잡한 요소도 좀더 부딪혀봐야 실력이 늘것같다.
- 회원가입 - 불가능한 폼 : 빨간박스, 가능한 폼 : 초록 박스 및 문구

- 개선된 프론트 실력
	- [[4. Object#편의 객체|세션]]을 객체로 가져올 수 있다.


