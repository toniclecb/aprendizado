# Spring Security | Curso 2022
Michelli Brito - https://www.youtube.com/watch?v=t6prPki7daU

**Autenticação e controle de acesso**

**Dependência**: spring-boot-starter-security

## Basic Authentication
HTTP Request: Header Authorization Baasic + encode username:password

Códigos Http: 401 Unauthorized e 401 Forbidden

Após adicionar a dependência e subir a aplicação é logado a linha:

`Using generated security password: 412x216512-5821s-4512-351ds3fa1s5d`

É uma credencial padrão do Spring, essa credencial será validada nos filtros dos Spring que gerenciam as requisições e avaliam a parte de autenticação, já o usuário padrão é "user".
Após adição da do Spring Security os endpoints da aplicação retornam 401 e agora necessitam de Basic Auth para funcionar. O Basic Auth é o tipo de autenticação padrão ao adicionar o projeto Security.

### Configurando HttpSecurity

Seguindo vamos configurar o projeto para usar autenticação. **Existe uma forma (em componentes) mais atual para configurar o Spring, veremos depois!**

Classe de configuração da autenticação:
```java
@Configuration // informa ao Spring que a classe é um Bean de configuração
        public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    	// WebSecurityConfigurerAdapter é o Adapter que configura a autenticação do Spring
    	@Override
        protected void configure(HttpSecurity http) throws Exception {
            http
                    .httpBasic()
                    .and()
                    .authorizeHttpRequests()
                    .anyRequest().permitAll();
        }
```
Neste caso, mesmo com o Security todas as requisições foram liberadas por causa do permitAll(), se alterarmos para authenticated() todas as requisições precisaram de autenticação.
**@EnableWebSecurity:** desliga todas as configurações padrões do Spring Security
Aqui ainda não tem nenhum controle de acesso, portanto um código 403 será retornado caso tente se usar verbo POST, por exemplo. Isso por que o **CSRF** vem habilitado no Security! O código abaixo desabilita essa configuração. Faz sentido desabilitar se sua API for utilizada por usuários fora de um navegador (uma API).
`.and().csrf().disable();`

### Configurando AuthenticationManagerBuilder
```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
	auth.inMemoryAuthentication()
		.withUser("michelli")
		.password(passwordEncoder().encode("607080"))
		.roles("ADMIN");
}

@Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
```
Depois dessa configuração o Spring não vai mais gerar a credencial ao iniciar o projeto.

### Configurando AuthenticationManagerBuilder com usuários do banco

O Spring Security tem um modelo para usuários: **UserDetails** (com vários atributos que são utilizados no processo de segurança). Essa classe precisa ser implementada no seu modelo de Usuário e os métodos precisam ser implementados.

`public class UserModel implements UserDetails, Serializable {`

Precisamos criar um Service que buscará os usuários pelo nome de usuário:
```java
@Service
@Transactional // quando buscar os dados do banco vai ter acesso aos roles que ficam em outra tabela relacionada
public class UserDetailsServiceImpl implements UserDetailsService {

    final UserRepository userRepository; // utilizamos nosso repository

    public UserDetailsServiceImpl(UserRepository userRepository) { // injeção do nosso repository
        this.userRepository = userRepository;
    }

    @Override // método da classe UserDetailsService que precisa ser implementado
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        UserModel userModel = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("User Not Found with username: " + username));
				// return userModel;
        return new User(userModel.getUsername(), userModel.getPassword(), true, true, true,true, userModel.getAuthorities());
    }
}
```
**UserDetailsService** é a classe do Spring Security responsável por prover um método que deve buscar os usuários pelo "username".
Precisamos agora mudar a configuração de autenticação (AuthenticationManagerBuilder):
```java
@Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService) // utilizando nosso Service
                .passwordEncoder(passwordEncoder());

    }
```

## Definindo as roles

As roles podem ficar salvas no banco, logo RoleModel, RoleRepository, etc.

`public class RoleModel implements GrantedAuthority, Serializable {`

No mesmo formato de UserDetails o Spring utiliza o **GrantedAuthority** para o modelo de autorização.
As roles na aplicação podem ficar em um Enum.

`private List<RoleModel> roles; // isso vai ser mapeado para uma nova tabela de relacionamento @ManyToMany`

Vamos restringir os acessos aos endpoints:
```java
@Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .httpBasic()
                .and()
                .authorizeHttpRequests()
                .antMatchers(HttpMethod.GET, "/parking-spot/**").permitAll()
                .antMatchers(HttpMethod.POST, "/parking-spot").hasRole("USER")
                .antMatchers(HttpMethod.DELETE, "/parking-spot/**").hasRole("ADMIN")
                .anyRequest().authenticated()
                .and()
                .csrf().disable();
    }
```
O que as linhas acima fazem:
- todos os métodos GET são permitidos;
- os métodos POST, apenas USER pode acessar;
- os métodos DELETE, apenas ADMIN pode acessar.
outro formato possível:

`.antMatchers(HttpMethod.POST, "/parking-spot").hasAnyRole("USER", "ADMIN")`

## Componentes ao invés de WebSecurityConfigurerAdapter
Spring Security 5.7+

WebSecurityConfig será alterada!
Devemos agora configurar o HttpSecurity em filterChain().

```java
@Configuration
public class WebSecurityConfigV2 {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
                .httpBasic()
                .and()
                .authorizeHttpRequests()
                .antMatchers(HttpMethod.GET, "/parking-spot/**").permitAll()
                .antMatchers(HttpMethod.POST, "/parking-spot").hasRole("USER")
                .antMatchers(HttpMethod.DELETE, "/parking-spot/**").hasRole("ADMIN")
                .anyRequest().authenticated()
                .and()
                .csrf().disable();
        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```
Essa nova maneira facilita em dois pontos pois por baixo dos panos o Security encontra dois beans sem a necessidade de explicitar no código (passwordEncoder() e UserDetailsServiceImpl), não precisa criar as injeções!

## Outra forma de controle de acesso
Isso quer dizer que não precisamos das linhas abaixo, comente elas:
```java
                //.antMatchers(HttpMethod.GET, "/parking-spot/**").permitAll()
                //.antMatchers(HttpMethod.POST, "/parking-spot").hasRole("USER")
                //.antMatchers(HttpMethod.DELETE, "/parking-spot/**").hasRole("ADMIN")
```
e anotar a classe WebSecurityConfigV2 com **EnableGlobalMethodSecurity**.
```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class WebSecurityConfigV2 {
```
Essa anotação significa que o controle deverá estar dentro dos Controllers.
Segue os métodos do Controller anotados:
```java
@PreAuthorize("hasRole('ROLE_ADMIN')")
    @PostMapping
    public ResponseEntity<Object> saveParkingSpot(@RequestBody @Valid ParkingSpotDto parkingSpotDto){
	...

@PreAuthorize("hasAnyRole('ROLE_ADMIN', 'ROLE_USER')")
    @GetMapping
    public ResponseEntity<Page<ParkingSpotModel>> getAllParkingSpots(
		...

@PreAuthorize("hasAnyRole('ROLE_ADMIN', 'ROLE_USER')")
    @GetMapping("/{id}")
    public ResponseEntity<Object> getOneParkingSpot(@PathVariable(value = "id") UUID id){

@PreAuthorize("hasRole('ROLE_ADMIN')")
    @DeleteMapping("/{id}")
    public ResponseEntity<Object> deleteParkingSpot(@PathVariable(value = "id") UUID id){
...

    @PreAuthorize("hasRole('ROLE_ADMIN')")
    @PutMapping("/{id}")
    public ResponseEntity<Object> updateParkingSpot(@PathVariable(value = "id") UUID id,
...
```