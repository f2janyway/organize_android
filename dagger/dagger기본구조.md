2020-05-11
=
1. Component
   1. 만약 모듈이 있다면(modules =[CarModule::class,...]) 이런 모양
   2. subComponent를 이용할 때 저 모듈을 이용.
   3. 한 프로젝트에 한 Component 만 허용
   4. Component 와 subCompnent 의 scope 가 겹치면 안됨(annotation 이름이)
   5. @Component.Factory or @Component.Build 가능 ( or !!!! 이다. 중복 안됨)
   6. 이런 기본적인 특징은 subcomponent 도 같다.

2. Module
   1. 모듈에는 scope를 넣지 않음.
   2. @Provides : static or public class (fun) or @Binds : abstract class ( interface)(fun)를 이용함
```
//Component

@Singleton
@Component(modules = [ApplicationModule::class,SubComponentModule::class])
interface ApplicationComponent{

    fun inject(ap: MyApplication)

    fun carComponent():CarComponent.Factory


    @Component.Builder
    interface Builder{

        @BindsInstance
        fun myApplication(myApplication: MyApplication):Builder

        fun build():ApplicationComponent

    }
}
```

```
@Module
class ApplicationModule  {
    @Singleton
    @Provides
    fun provideSharedPref(myApp:MyApplication): SharedPreferences =
        PreferenceManager.getDefaultSharedPreferences(myApp)
}
```

```
@Module(subcomponents = [CarComponent::class])
class SubComponentModule
```

```
@ActivityScope
@Subcomponent(modules = [CarModule::class])
interface CarComponent {

    fun inject(m: MainActivity)

    @Subcomponent.Factory
    interface Factory {
        fun create(
            @BindsInstance @Named("radius")radius: Int,
            @BindsInstance @Named("pressure")pressure: Int
        ): CarComponent
    }

}
```
# module
static 방식
```

@Module
object CarModule {
    @JvmStatic
    @Provides
    @Binds
    fun provideCar(wheel: Wheel, dieselEngine: DieselEngine) = Car(wheel, dieselEngine)

    @JvmStatic
    @Provides
    fun provideWheel(rim: Rim, tire: Tire) = Wheel(rim, tire)

    @JvmStatic
    @Provides
    fun provideRim(@Named("radius") radius: Int) = Rim(radius)

    @JvmStatic
    @Provides
    fun provideTire(@Named("pressure") pressure: Int) = Tire(pressure)

    @JvmStatic
    @Provides
    fun provideEngine() = DieselEngine()
}
```
public
```
@Module
class CarModule {
    @Provides
    fun provideCar(wheel: Wheel, dieselEngine: DieselEngine) = Car(wheel, dieselEngine)

    @Provides
    fun provideWheel(rim: Rim, tire: Tire) = Wheel(rim, tire)

    @Provides
    fun provideRim(@Named("radius") radius: Int) = Rim(radius)

    @Provides
    fun provideTire(@Named("pressure") pressure: Int) = Tire(pressure)

    @Provides
    fun provideEngine() = DieselEngine()
}
```

abstract 이해하기