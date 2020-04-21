# Tutorial

Before we start this tutorial, please make sure you have installed Xcode (v11 is preferred) on your Mac.  
After this tutorial, you might get a deep understanding of ObjCUI.

## Setup for the tutorial

First of all, please create a "Single View Application" in Xcode, then we are ready to install ObjCUI as the project dependency.

### Install by Cocoapods
Cocoapods is preferred for managing dependencies of iOS projects. You can use it by typing command:

```bash
# in your project directory
pod init 
```

Then change the content of given `Podfile`:

```ruby
target 'YourApp' do
    pod 'ObjCUI', :git=> 'https://github.com/ObjCUI/ObjCUI.git', :tag => '0.0.1'
end
```

### Install manually
If you don't use Cocoapods to manage your dependencies, we also provide the raw binary for your manual installing.
Just download releases of ObjCUI repositry, find the `ObjCUI.framework` and drag it into your application target "Embed Binary and Libraries".

## Overview
Now let us get started.

### What is ObjCUI?

ObjCUI is a declarative, efficient Objective-C library for building CocoaTouch user interfaces. It lets you compose complex UIs from small and isolated pieces of code called "components".  

ObjCUI provided few kinds of builtin components as examples, but all of them are derived from `UIView` or `ObjCUIComponent`. So you can just use these components to compose user interface, and you can also implement custom component from your existing widget.

For example:  

```Objective-C
@interface ExampleComponent: ObjCUIComponent

@property (nonatomic) id imageProps;

@end

@implementation ExampleComponent

- (ObjCUIElement *)render {
    return (
        View(
            ImageView().props(self.imageProps),
            View(),
            nil
        );
    );
}

@end

// exmaple usage:
// [ObjcUI mount:ExampleComponent() onView:self.view];

```

We'll get funny view decalration in code block syntax, each code block will instance a view component. We use components to tell ObjCUI what we want to display on the screen. When component data changes, ObjCUI will automatically update and re-render the changes.

### What we will do
We'll create a view looks like a card with user avatar, name and other informations. Then we download data from the remote server and changes these contents.

### Create your first component
In your project, create a CocoaTouch `Class` declaration, and use `ObjCUIComponent` as its superclass, to make it easy to delcare, ObjCUI provide a macro called `OBJCUI_DECLARE_SINGLECHILD_ELEMENT` to do this.

```Objective-C
@import ObjCUI;

OBJCUI_DECLARE_SINGLECHILD_ELEMENT(Foo)

@interface FooComponent : ObjCUIComponent
@end

```

#### implement the render 
We want a card to wrap avatar image view, name label, or any other views, so the render method should provide a root view as the container. Then we add avatar image view, labels into this view.


```Objective-C
@implementation FooComponent

- (ObjCUIElement *)render {
    return View(        // the root container
        ImageView(),    // the avatar
        View(           // right container
            Label(),    // name label
            Label(),    // gender label
            nil
        ),
        nil
    );
}

@end

```

#### passing data through props

Each view component will receive an object call props to fill its content. So we add some props getter here to passing them into vieww components.

```Objective-C
@interface FooComponent ()

@property (nonatomic, readonly) ObjcUIImageViewProp *imageProps;
@property (nonatomic, readonly) ObjcUILabelProp *nameProps;
@property (nonatomic, readonly) ObjcUILabelProp *genderProps;

@end

@implementation FooComponent

- (ObjcUIElement *)render {
    return (
        View(
             ImageView().props(self.imageProps),
             View(
                  Label().props(self.nameProps),
                  Label().props(self.genderProps),
                  nil),
             nil)
    );
}

@end
```

#### accessing data from current scope
We have declared passing props into image view and labels, and now we can retrive these data from props in current component scope.

```Objective-C
// the type of FooComponent's props is a NSDictionary, you can also change it into other typs.

- (ObjcUIImageViewProp *)imageProps {
    ObjcUIImageViewProp *props = ObjcUIImageViewProp.new;
    props.image = [self.props objectForKey:@"avatar"];
    props.cornerRadius = @50;
    props.clipToBounds = @(YES);
    props.borderColor = UIColor.grayColor;
    props.borderWidth = @2.f;
    return props;
}

- (ObjcUILabelProp *)nameProps {
    ObjcUILabelProp *label = ObjcUILabelProp.new;
    label.text = [self.props objectForKey:@"name"];
    return label;
}

- (ObjcUILabelProp *)genderProps {
    ObjcUILabelProp *label = ObjcUILabelProp.new;
    label.text = [self.props objectForKey:@"gender"];
    return label;
}

```


#### styling components
After filling data, we should also styling these components to display them.  
ObjCUI used YogaKit to make it easy to styling components, this means Yogakit is a required dependency of ObjCUI.

```Objective-C
- (ObjcUIElement *)render {
    return (
        View(
             ImageView().props(self.imageProps)
             .styles(ObjcUIStyleBuilder(^(ObjcUIStyleType * style) {
                style.width = YGPointValue(100);
                style.height = YGPointValue(100);
            })),

             View(
                  Label().props(self.nameProps),
                  Label().props(self.genderProps),
                  nil)
                .styles(ObjcUIStyleBuilder(^(ObjcUIStyleType *style) {
                    style.flexGrow = 1.f;
                    style.flexDirection = YGFlexDirectionColumn;
                    style.alignItems = YGAlignStretch;
                }))                  
             nil)
            .styles(ObjcUIStyleBuilder(^(ObjcUIStyleType *style) {
                style.width = YGPointValue(300);
                style.height = YGPointValue(200);
                style.flexDirection = YGFlexDirectionRow;
            }))
    );
}

```

#### mount the component
We have almost completed all of the views we need, at last we need to mount the `Foo` component into some view.

```Objective-C
// in UIViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    ObjCUIElement *foo = Foo().props(@{@"name": @"foo", @"gender": @"bar"});
    [ObjcUI mountComponent:foo
                    onView:self.view];
}
```

#### changing passed data
Data is not stateful outside ObjCUI components, so we need to change it manually when conditions occurred.

```Objective-C
// mocking network latency
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
    foo.props(@{
        @"avatar": [UIImage imageNamed:@"avatar"],
        @"name": @"nook",
        @"gender": @"man"
    });
});
```
