
@RunWith(SpringRunner.class)
@WebMvcTest(UserController.class)
public class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private AuthService authService;

    @MockBean
    private UserService userService;

    @Test
    public void testLogin_Success() throws Exception {
        // Mock AuthService
        Mockito.when(authService.authenticate("testUser", "password123")).thenReturn(true);

        // 模拟请求数据
        String loginRequest = """
            {
                "username": "testUser",
                "password": "password123"
            }
            """;

        // 发起 POST 请求
        mockMvc.perform(post("/api/user/login")
                .contentType(MediaType.APPLICATION_JSON)
                .content(loginRequest))
                .andExpect(status().isOk())
                .andExpect(content().string("Login successful"));
    }

    @Test
    public void testLogin_Failure() throws Exception {
        // Mock AuthService
        Mockito.when(authService.authenticate("testUser", "wrongPassword")).thenReturn(false);

        String loginRequest = """
            {
                "username": "testUser",
                "password": "wrongPassword"
            }
            """;

        mockMvc.perform(post("/api/user/login")
                .contentType(MediaType.APPLICATION_JSON)
                .content(loginRequest))
                .andExpect(status().isUnauthorized())
                .andExpect(content().string("Invalid credentials"));
    }

    @Test
    public void testGetUserInfo_Success() throws Exception {
        // Mock UserService
        UserInfo mockUserInfo = new UserInfo("1", "testUser", "test@example.com");
        Mockito.when(userService.getUserInfo("1")).thenReturn(mockUserInfo);

        mockMvc.perform(get("/api/user/1")
                .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.userId").value("1"))
                .andExpect(jsonPath("$.username").value("testUser"))
                .andExpect(jsonPath("$.email").value("test@example.com"));
    }

    @Test
    public void testGetUserInfo_NotFound() throws Exception {
        // Mock UserService
        Mockito.when(userService.getUserInfo("2")).thenReturn(null);

        mockMvc.perform(get("/api/user/2")
                .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isNotFound());
    }
}
