# 问题集合

1，反序列化与getter和setter以及isXxx方法有关，如果存在个getXxx的方法，但是没有对应Xxx属性就会出现序列化问题，序列化时应该忽视该方法

**注意：** getXxx 的返回属性，必须和Xxx的属性类型一致，不然报错。

* **@JsonIgnoreProperties(ignoreUnknown = true)**
*  **@JsonIgnore**

```java

@Data
@JsonIgnoreProperties(ignoreUnknown = true)/*********** 忽视isXxx的属性 ***********/
public class LoginUser implements UserDetails, Serializable {
    private static final long serialVersionUID = 1L;


    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getUserName();
    }

    /**
     * 账户是否未过期
     */
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    /**
     * 账户是否未锁定
     */
    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    /**
     * 用户凭证（密码）是否未过期
     */
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    /**
     * 是否可用
     */
    @Override
    public boolean isEnabled() {
        return true;
    }

    /**
     * 用户权限
     */
    @Override
    @JsonIgnore /*********** 忽视该该方法 *************/
    public Collection<? extends GrantedAuthority> getAuthorities() {
        ArrayList<GrantedAuthority> authorityList = new ArrayList<>();
        for (String permission : permissions) {
            if (!StringUtils.isEmpty(permission)) {
                SimpleGrantedAuthority authority = new SimpleGrantedAuthority(permission);
                authorityList.add(authority);
            }
        }
        return authorityList;
    }
}

```

