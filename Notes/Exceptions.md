### Исключения

#### Перехват

```d
void main()
{
    try
    {
        throw new StringException("You shall not pass.");
    }
    catch (FileException e)
    {
        // ...
    }
    catch (StringException e)
    {
        // ...
    }
    finally
    {
        // ...
    }    
}
```

#### Пользовательские исключения

```d
class UserNotFoundException: Exception
{
    this(string msg, string file = __FILE__, size_t line = __LINE__) {
            super(msg, file, line);
        }
}

void main()
{
    throw new UserNotFoundException("D-Man is on vacation");
}
```

#### Функции nothrow

```d
bool lessThan(int left, int right) nothrow
{
    return left < right;
}
```
