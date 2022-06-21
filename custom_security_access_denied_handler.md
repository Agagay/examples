# Symfony 5+. Access denied handler без переопределения шаблонов
Если нужно переопределить страницу с ошибкой и отдавать ее при не авторизованном доступе, достаточно сделать по доке https://symfony.com/doc/current/security/access_denied_handler.html#customize-the-unauthorized-response

В данном примере рассмотрера проблема, при которой не надо переопределять стандартные шаблоны с ошибками. 
Нужно выдавать вместо стандартного шаблона 403, стандартный шаблон 404

## Хендлер
```php
// /src/Security/AccessDeniedHandler.php
namespace App\Security;

use Symfony\Component\ErrorHandler\ErrorRenderer\ErrorRendererInterface;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;
use Symfony\Component\Security\Core\Exception\AccessDeniedException;
use Symfony\Component\Security\Http\Authorization\AccessDeniedHandlerInterface;

class AccessDeniedHandler implements AccessDeniedHandlerInterface
{
    private ErrorRendererInterface $errorRenderer;

    public function __construct(ErrorRendererInterface $errorRenderer)
    {
        $this->errorRenderer = $errorRenderer;
    }

    public function handle(Request $request, AccessDeniedException $accessDeniedException): ?Response
    {
        $exception = new NotFoundHttpException();
        $exception = $this->errorRenderer->render($exception);

        return new Response($exception->getAsString(), $exception->getStatusCode(), $exception->getHeaders());
    }
}
```

## Передать HtmlErrorRenderer в AccessDeniedHandler
```yaml
# /config/service.yaml
    App\Security\AccessDeniedHandler:
        arguments:
            $errorRenderer: '@error_handler.error_renderer.html'
```

## Включить AccessDeniedHandler для main security.firewalls.main
```yaml
# /config/packages/security.yaml
security:
  ...
    firewalls:
        main:
            access_denied_handler: App\Security\AccessDeniedHandler
```
