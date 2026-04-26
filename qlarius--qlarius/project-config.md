---
trigger: always_on
description: Testing Best Practices
---


# Testing Best Practices

## General Testing Principles

### Test Structure
- Follow the AAA pattern: Arrange, Act, Assert
- One assertion concept per test (can have multiple assert statements for the same concept)
- Use descriptive test names that explain the behavior being tested
- Group related tests using `describe` blocks
- Keep tests independent and isolated

```elixir
# ✅ Good: Clear test structure
describe "create_post/2" do
  test "creates a post with valid attributes" do
    # Arrange
    user = user_fixture()
    attrs = %{title: "Test", content: "Content"}
    
    # Act
    assert {:ok, %Post{} = post} = Blog.create_post(user, attrs)
    
    # Assert
    assert post.title == "Test"
    assert post.content == "Content"
    assert post.user_id == user.id
  end

  test "returns error changeset with invalid attributes" do
    user = user_fixture()
    assert {:error, %Ecto.Changeset{}} = Blog.create_post(user, %{})
  end
end
```

## Unit Testing

### Context Testing
- Test all public functions in context modules
- Test both happy paths and error cases
- Test validation rules
- Test business logic edge cases
- Mock external dependencies (HTTP calls, third-party APIs)

```elixir
# ✅ Good: Comprehensive context tests
defmodule Qlarius.AccountsTest do
  use Qlarius.DataCase

  alias Qlarius.Accounts

  describe "get_user/1" do
    test "returns user when id exists" do
      user = user_fixture()
      assert Accounts.get_user(user.id) == user
    end

    test "returns nil when id doesn't exist" do
      assert Accounts.get_user(Ecto.UUID.generate()) == nil
    end
  end
end
```

### Schema and Changeset Testing
- Test all validations
- Test unique constraints
- Test associations
- Test custom changeset functions

```elixir
# ✅ Good: Changeset validation tests
describe "user changeset" do
  test "validates email format" do
    changeset = User.changeset(%User{}, %{email: "invalid"})
    assert "has invalid format" in errors_on(changeset).email
  end

  test "validates email uniqueness" do
    user_fixture(email: "test@example.com")
    {:error, changeset} = Accounts.create_user(%{email: "test@example.com"})
    assert "has already been taken" in errors_on(changeset).email
  end
end
```

## Integration Testing

### Controller Testing
- Test all controller actions
- Test authentication and authorization
- Test error handling
- Test redirects and flash messages

```elixir
# ✅ Good: Controller test
describe "create post" do
  test "redirects to show when data is valid", %{conn: conn} do
    conn = post(conn, Routes.post_path(conn, :create), post: valid_attrs())
    assert %{id: id} = redirected_params(conn)
    assert redirected_to(conn) == Routes.post_path(conn, :show, id)
  end

  test "renders errors when data is invalid", %{conn: conn} do
    conn = post(conn, Routes.post_path(conn, :create), post: invalid_attrs())
    assert html_response(conn, 200) =~ "New Post"
  end
end
```

### LiveView Testing
- Use `Phoenix.LiveViewTest` helpers
- Test mount, handle_event, handle_params
- Test form submission and validation
- Test PubSub interactions
- Use `has_element?/2` instead of raw HTML checks

```elixir
# ✅ Good: LiveView test
defmodule QlariusWeb.PostLive.IndexTest do
  use QlariusWeb.ConnCase
  import Phoenix.LiveViewTest

  describe "Index" do
    test "displays all posts", %{conn: conn} do
      post = post_fixture(title: "Test Post")
      {:ok, view, _html} = live(conn, ~p"/posts")
      
      assert has_element?(view, "#posts")
      assert has_element?(view, "#post-#{post.id}")
      assert render(view) =~ "Test Post"
    end

    test "creates new post on form submit", %{conn: conn} do
      {:ok, view, _html} = live(conn, ~p"/posts/new")
      
      assert view
             |> form("#post-form", post: valid_attrs())
             |> render_submit()
      
      assert_redirected(view, ~p"/posts/#{post_id}")
    end
  end
end
```

## Test Data Management

### Fixtures
- Use factories or fixtures for test data
- Keep fixtures in `test/support/fixtures`
- Make fixtures flexible with optional overrides
- Avoid database operations in fixtures when possible

```elixir
# ✅ Good: Flexible fixture
def user_fixture(attrs \\ %{}) do
  attrs =
    Enum.into(attrs, %{
      email: "user-#{System.unique_integer()}@example.com",
      name: "Test User"
    })

  {:ok, user} = Accounts.create_user(attrs)
  user
end
```

### Database Testing
- Use `Qlarius.DataCase` for tests that need the database
- Tests run in transactions (rolled back automatically)
- Use `Ecto.Adapters.SQL.Sandbox` for concurrent tests
- Set `pool: Ecto.Adapters.SQL.Sandbox` in test config

```elixir
# ✅ Good: DataCase setup
defmodule Qlarius.DataCase do
  use ExUnit.CaseTemplate

  using do
    quote do
      alias Qlarius.Repo
      import Ecto
      import Ecto.Changeset
      import Ecto.Query
      import Qlarius.DataCase
    end
  end

  setup tags do
    Qlarius.DataCase.setup_sandbox(tags)
    :ok
  end
end
```

## Async Testing

### Concurrent Tests
- Use `async: true` for tests that don't share state
- Significantly speeds up test suite
- Don't use async for tests that:
  - Modify global state
  - Use shared resources
  - Have side effects outside the database

```elixir
# ✅ Good: Async tests when possible
defmodule Qlarius.AccountsTest do

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Qlarius) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
