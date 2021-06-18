# Symfony 5 - Build your first API

## Lời nói đầu

Symfony là một trong những framework PHP hàng đầu, được xây dựng dữa trên những thành phấn có thể tái sử dụng. Các framework khác như Drupal, Laravel, workpress đều phải dùng đến các thành phần của symfony này. Bạn nào làm nhiều với Laravel chắc hẳn thấy rất rõ điều này, mọi class trong laravel hầu như đều extend từ symfony mà ra.

Đã lâu rồi k động tới symfony, từ ngay còn làm với version 1.4 tới giờ (version mới nhất là 5.3), hôm nay có thời gian rảnh rỗi ngồi xem lại trong vài năm vừa qua symfony có sự thay đổi như thế nào. Nếu bạn quan tâm, thì cùng tôi đi hết bài viết này nhé

Thay vì ngồi đọc tài liệu, thì chúng ta sẽ bắt tay vào code luôn một ứng dụng web đơn giản sử dụng symfony 5 nhé. Dựa trên bài toán là phát triển một ứng dụng web có tích hợp thêm authentication. Trước tiên, hãy thử với symfony Guard, sau khi thành thục rồi thì chúng ta sẽ compare nó với Auth0. 

## Bắt đầu

Chúng ta sẽ xây dựng một danh sách các công ty công nghệ đơn giản, mục đích của bài viết chỉ là đăng nhập và vượt qua được bước authentication, nên các bạn cũng k cần đòi hỏi một bài toán gì quá phức tạp. Trước tiên, chúng ta cần tạo 1 project symfony, với câu lệnh

```
composer create-project symfony/website-skeleton top-tech-companies
```

Sau khi command này chạy xong, bạn sẽ thấy một thư mục mới được tạo ra với tên _top-tech-companies_ trong root folder, nơi bạn chạy command trên. Nó cũng cài luôn cho bạn các required dependencies, ví dụ như: 

```
- symfony/maker-bundle: Package này giúp bạn tạo các controller, models, các class test, ...

- symfony/security-bundle: Tích hợp bảo mật hoàn chỉnh cho ứng dụng symfony

- symfony/flex: Giúp thêm các tính năng mới một cách liên mạch
```

Trong bài viết này câu lệnh symfony/maker-bundle sẽ được sử dụng rất nhiều đấy.

Có cách khác để tạo mới một project symfony, đó là dùng Symfony installer, các bạn tự tìm hiểu nhé

## Cấu trúc thư mục.

Trước khi đi vào code, hãy xem qua cấu trúc thư mục project của chúng ta vừa tạo ra đã nhé.

```
your-project/
├─ bin/
│  ├─ console
│  └─ phpunit
├─ config/
│  └─ packages/
│  └─ routes/
├─ public/
│  └─ index.php
├─ src/
│  └─ Controller/
│  └─ Entity/
│  └─ Form/
│  └─ Migrations/
│  └─ Repository/
│  └─ Security/
│  └─ Kernel.php
├─ templates/
├─ translations/
├─ var/
├─ vendor
   └─ ...
```

Giải thích qua 1 chút nhé:

```
- bin: Chứa các file thực thi
- config: chứa tất cả các file config của dự án
- public: đây là thư mục document root
- index.php: file này được gọi tới khi apache/ nginx của bạn được chạy
- src: Chưa toàn bộ code controller, service
- templates: Chưa các file view. twig
- translations: hỗ trợ đa ngôn ngữ
```

## Khởi chạy ứng dụng

Trong thư mục dự án, chạy câu lệnh

```
// Change directory
cd top-tech-companies

// install web server
composer require symfony/web-server-bundle --dev ^4.4.2
```

Tiếp theo là chạy câu lệnh: 

```
php bin/console server:run
```
Bây giờ bạn hãy truy cập vào link [http://localhost:8000](http://localhost:8000) trên trình duyệt của mình để kiểm tra nhé.

##. Tạo User Class

Hiểu đơn giản thì chúng ta sẽ tạo một User class hoặc 1 Entity, để có thể đăng ký hoặc authentication một user vào hệ thống, sử dụng lệnh

```
php bin/console make:user
```

Command này sẽ hỏi bạn có muốn tạo thêm các file liên quant tới user class hay k, nếu k hiểu thì bạn hãy chọn yes hết nhé

Sau khi câu lệnh hoàn tất, nó sẽ tạo ra các file mới là src/Entity/User.php và src/Repository/UserRepository.php, nó cũng update file config/packages/security.yaml. Thông tin chi tiết tôi sẽ nói ở phần sau. 

Đầu tiên, chúng ta cần add thêm 1 số trường mới vào trong file User.class

```
// src/Entity/User.php
<?php

namespace App\Entity;
use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Security\Core\User\UserInterface;
/**
 * @ORM\Entity(repositoryClass="App\Repository\UserRepository")
 */
class User implements UserInterface
{
    /**
     * @ORM\Id()
     * @ORM\GeneratedValue()
     * @ORM\Column(type="integer")
     */
    private $id;

    /**
     * @ORM\Column(type="string", length=180, unique=true)
     */
    private $email;

    /**
     * @ORM\Column(type="json")
     */
    private $roles = [];

    /**
     * @var string The hashed password
     * @ORM\Column(type="string")
     */
    private $password;

    /**
     * @ORM\Column(type="string", length=255)
     */
    private $name;

    public function getId(): ?int
    {
        return $this->id;
    }

    public function getEmail(): ?string
    {
        return $this->email;
    }

    public function setEmail(string $email): self
    {
        $this->email = $email;
        return $this;
    }

    /**
     * A visual identifier that represents this user.
     *
     * @see UserInterface
     */
    public function getUsername(): string
    {
        return (string) $this->email;
    }

    /**
     * @see UserInterface
     */
    public function getRoles(): array
    {
        $roles = $this->roles;
        // guarantee every user at least has ROLE_USER
        $roles[] = 'ROLE_USER';
        return array_unique($roles);
    }

    public function setRoles(array $roles): self
    {
        $this->roles = $roles;
        return $this;
    }

    /**
     * @see UserInterface
     */
    public function getPassword(): string
    {
        return (string) $this->password;
    }

    public function setPassword(string $password): self
    {
        $this->password = $password;
        return $this;
    }

    /**
     * @see UserInterface
     */
    public function getSalt()
    {
        // not needed when using the "bcrypt" algorithm in security.yaml
    }

    /**
     * @see UserInterface
     */
    public function eraseCredentials()
    {
        // If you store any temporary, sensitive data on the user, clear it here
        // $this->plainPassword = null;
    }

    public function getName(): ?string
    {
        return $this->name;
    }

    public function setName(string $name): self
    {
        $this->name = $name;
        return $this;
    }
}
```

Với việc sử dụng Symfony MakerBundle, khi chúng ta thêm một thuộc tính name mới, thì nó sẽ tự động sinh ra cho chúng ta 2 function get và set. Khá là tiện lợi phải k 

## Cài đặt Controllers

Mô hình MVC thì controller k thể thiếu, để tạo controller bạn dùng câu lệnh

```
php bin/console make:controller ListControlle
```

Câu lệnh trên sẽ tạo ra cho bạn 2 file là _src/Controller/ListController.php_ và _templates/list/index.html.twig_ . Mở file ListController.php và thêm nội dung sau

```
// ./src/Controller/ListController
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Routing\Annotation\Route;

class ListController extends AbstractController
{
    /**
     * @Route("/list", name="list")
     */
    public function index(Request $request)
    {
        $companies = [
            'Apple' => '$1.16 trillion USD',
            'Samsung' => '$298.68 billion USD',
            'Microsoft' => '$1.10 trillion USD',
            'Alphabet' => '$878.48 billion USD',
            'Intel Corporation' => '$245.82 billion USD',
            'IBM' => '$120.03 billion USD',
            'Facebook' => '$552.39 billion USD',
            'Hon Hai Precision' => '$38.72 billion USD',
            'Tencent' => '$3.02 trillion USD',
            'Oracle' => '$180.54 billion USD',
        ];

        return $this->render('list/index.html.twig', [
            'companies' => $companies,
        ]);
    }
}

```
Để mọi việc đơn giản và nhanh chóng hơn, hiện tại tôi đang hard code danh sách các companies và truyền xuống file html. Tiếp theo, hãy tạo file controller để xử lý đăng ký mới của user

```
php bin/console make:controller RegistrationController
```
Tương tự, chúng ta cũng có 2 file mới đc sinh ra, file controller và file html. Mở file controller và thêm nội dung

```
// ./src/Controller/RegistrationController
<?php

namespace App\Controller;

use App\Entity\User;
use App\Form\UserType;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Routing\Annotation\Route;
use Symfony\Component\Security\Core\Encoder\UserPasswordEncoderInterface;

class RegistrationController extends AbstractController
{
    private $passwordEncoder;

    public function __construct(UserPasswordEncoderInterface $passwordEncoder)
    {
        $this->passwordEncoder = $passwordEncoder;
    }

    /**
     * @Route("/registration", name="registration")
     */
    public function index(Request $request)
    {
        $user = new User();

        $form = $this->createForm(UserType::class, $user);

        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            // Encode the new users password
            $user->setPassword($this->passwordEncoder->encodePassword($user, $user->getPassword()));

            // Set their role
            $user->setRoles(['ROLE_USER']);

            // Save
            $em = $this->getDoctrine()->getManager();
            $em->persist($user);
            $em->flush();

            return $this->redirectToRoute('app_login');
        }

        return $this->render('registration/index.html.twig', [
            'form' => $form->createView(),
        ]);
    }
}
```

Lưu ý rằng, ở đây tôi đang định nghĩa router /registation tới hàm index của controller này. Khi bạn truy cập vào đường link có /registration trên URL, thì mọi xử lý sẽ được vào trong hàm index. Tiếp tới, hãy tạo một controller để handle logic login

```
php bin/console make:controller SecurityController
```

Trong controller registration nếu bạn để ý thì chúng ta đang gọi tới một User Type. chúng ta cần tạo ra nó để code k bị lỗi

```
php bin/console make:form
```
Khi câu hỏi hiện lên, hãy trả lời nó là UserType, việc còn lại chỉ là chờ câu lệnh chạy xong, Mở file src/Form/UserType.php và edit

```
// src/Form/UserType.php
<?php

namespace App\Form;

use App\Entity\User;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\EmailType;
use Symfony\Component\Form\Extension\Core\Type\PasswordType;
use Symfony\Component\Form\Extension\Core\Type\RepeatedType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class UserType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('email', EmailType::class)
            ->add('name', TextType::class)
            ->add('password', RepeatedType::class, [
                'type' => PasswordType::class,
                'first_options' => ['label' => 'Password'],
                'second_options' => ['label' => 'Confirm Password']
            ])
        ;
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults([
            'data_class' => User::class,
        ]);
    }
}
```

## Tạo giao diện đăng nhập

Sử dụng lệnh

```
php bin/console make:auth
```

Sau khi hoàn thành lệnh trên, bạn sẽ đc file src/Security/LoginFormAuthenticator.php and templates/security/login.html.twig. Tiếp theo mở controller và edit

```
// src/Controller/SecurityController.php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;
use Symfony\Component\Security\Http\Authentication\AuthenticationUtils;

class SecurityController extends AbstractController
{
    /**
     * @Route("/", name="app_login")
     */
    public function login(AuthenticationUtils $authenticationUtils): Response
    {
        // get the login error if there is one
        $error = $authenticationUtils->getLastAuthenticationError();
        // last username entered by the user
        $lastUsername = $authenticationUtils->getLastUsername();

        return $this->render('security/login.html.twig', ['last_username' => $lastUsername, 'error' => $error]);
    }

    /**
     * @Route("/logout", name="app_logout")
     */
    public function logout()
    {
        throw new \Exception('This method can be blank - it will be intercepted by the logout key on your firewall');
    }
}
```

Đến đây, ứng dụng của chúng ta đã cơ bản hoàn thành, bài sau tôi sẽ hướng dẫn các bạn cách config DB và s
