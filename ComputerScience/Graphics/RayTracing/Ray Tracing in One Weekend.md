## 1. Overview

Developing a ray tracer **incrementally** is a nice strategy to debug.

Here is the outline of what we do step by step to build the simple ray tracer.

1. Simple code to generate a plain text (may be the simplest **image** format) **PPM** file
2. Our first class **`vec3`**, which is the foundation of vectors and colors in our ray tracer
3. Create the **`ray`** class, the one thing that all ray tracers have; Then fix our simple **camera**, use **ray tracing** method to render **background**
4. Add our first **object**, a sphere, to the scene then render the image using a core step of ray tracing: find the **closest intersection** between a ray and the object, to render the image. The geometry is simple for **ray-sphere intersection**
5. Add **surface normal** for shading; Abstract **`hittable`** and **`hittable_list`** class for placing **multiple objects** in the sence and finding the **closest intersection** of a ray with the objects more easily
6. Use **multi-sampling** to do **antialiasing** in ray tracing, we need a **random number generator**
7. Use **material** to make our objects look more **realistic**. Here are 3 different implementations of **diffuse** materials with different **distributions**. Also use **gamma correction** to make the color looks brighter as in human eyes
8. Add more materials, abstract the **`material`** class, it describe **how light interact with the surface or object**, scatter, absorb and attenuate behavior of light. Then add **metal** material using **reflection**
9. **Dielectrics** material introducing **refraction** here. We can also model a **hollow class sphere** using dielectrics material
10. **Move**, **rotate** and **adjust** **_fov_** **of** our camera
11. Simulate **defocus blur** of real camera
12. Generate a cool image with more objects and materials!
13. Further exploration to make our ray tracer more powerful

## 2. Output an Image

My OS is Ubuntu 22.04, `feh` can be used to view _.ppm_ image:

```shell
feh <image-path>
```

````ad-help
 **\[Problem\]** ❓ When I try to recompile and rerun the program to generate _image.ppm_, I use redirect command:

```shell
./build/in1weekend > ./images/image.ppm
```

But I got an error message from bash:

> bash: ./images/image.ppm: cannot overwrite existing file

Then I found solution from [Why 'cannot overwrite existing file'?](https://superuser.com/questions/254644/why-cannot-overwrite-existing-file). It was because the `noclobber` option has been set.

**\[Solution\]** ✅ So I add:

```shell
set +o noclobber
```

to _~/.bashrc_ and _~/.bash_profile_ to turn off `noclobber` in my bash (need to reload the terminal or use `source` to make the change take effect)
````

````ad-note
title: **\[Extension\]** ⛏ `static_cast` & color conversion (Why 255.999)
We see the following code in color conversion:

```CPP
int ir = static_cast<int>(255.999 * r);
int ig = static_cast<int>(255.999 * g);
int ib = static_cast<int>(255.999 * b);
```

Here the double-to-int typecasting uses `static_cast<int>` instead of `(int)`.

The `(int)x` is C style typecasting where `static_cast(x)` is used in C++. This `static_cast<>()` gives **compile time checking facility**, but the C style casting does not support that.

Another thing makes me confusing is multiplying by 255.999 rather than 255.0 here. I found explaination in [Converting color value from float 0..1 to byte 0..255](https://stackoverflow.com/questions/1914115/converting-color-value-from-float-0-1-to-byte-0-255). 

Because in C++ it simply cut off the decimal part of the number when cast double to int and we want some number like 0.9999 also be casted to 255.
````

````ad-failure
title: **\[Fault\]** 🚧❓ divide by zero error
Here the code:

```CPP
auto r = double(i) / (image_width-1);
auto g = double(j) / (image_height-1);
```

do not check if the divisor is $0$, even though we use constant here.
````

## 3. The `vec3` Class

Here we need _two_ types of override for operator `*` because the scalar multiplication of vectors has **commutative** property.

```CPP
inline vec3 operator*(double t, const vec3 &v) {
    return vec3(t * v.e[0], t * v.e[1], t * v.e[2]);
}

inline vec3 operator*(const vec3 &v, double t) {
    return t * v;
}
```

While dividing vector by a scalar does not satisfy the commutative law, so here is only _one_ override.

## 4. Rays, a Simple Camera, and Background

At the core, the **ray tracer** _sends rays through pixels_ and _computes the color_ seen in the direction of those rays.

The involved steps are:

1. calculate the **ray** from the eye to the pixel
2. determine which objects the ray **intersects**
3. compute a **color** for that intersection point

To render an image using ray tracer, first set up the **pixel dimensions**, **virtual viewport** (they have the same aspect ratio) and the **focal length**. Then decide the **position of eye** and **coordinate system orientation** in the scene.

```ad-info
The **aspect ratio** of an image is the _ratio of its width to its height_. It is expressed as **width:height**
```

```ad-info
The **focal length** is the _distance_ between the _projection plane_ and the _projection point_
```

```ad-question
title: **\[Doubt\]** 🤔

Here:
 
- **Projection plane** refers to the sensor of the camera? Or on the plane which is symmetric to the sensor plane with respect to the view direction?
- **Projection point** refers to the eye (or the pinhole of the camera)?
```

Modify our code to render a blue-to-white gradient background top-down.

We should set up not only the **pixel dimensions** for the rendered image, but also a **virtual viewport** through which to pass our scene rays. For the standard square pixel spacing, _the viewport's aspect ratio should be the **same** as our rendered image_, otherwise the image will be _stretched_:

```CPP
// Image
const auto aspect_ratio = 16.0 / 9.0;
const int image_width = 400;
const int image_height = static_cast<int>(image_width / aspect_ratio);

// Camera
auto viewport_height = 2.0;
auto viewport_width = aspect_ratio * viewport_height;
auto focal_length = 1.0;
```


```ad-question
title: **\[Doubt\]** 🤔 

My understanding is that **pixel dimensions** for the image refers to the _"size" of the image measured in pixel_, and **virtual viewport** is a _small window we choose to "show" what we see in this "actual" world (combine it and focal length? together can define the **field of vision** for camera?)_.

Pixel dimensions should be **discrete** while virtual viewport should be **continuous**. We can adjust _how much detail (pixel dimensions) to display_ in the image without changing _what we see (viewport)_. Furthermore, the pixel dimensions are **sampling** of the viewport.
```

Here `ray_color(ray)` function is used to compute the color at the intersection point. The eye `(0, 0, 0)` are mapped to the center of this image along inverse z-axis, so add `1.0` to `unit_direction.y` (beacuse top-down) then times `0.5` to get `t` in `[0, 1]`(**normalization**). Finally **linearly blends** white and blue using `t`.

```CPP
color ray_color(const ray& r) {
    vec3 unit_direction = unit_vector(r.direction());
    auto t = 0.5*(unit_direction.y() + 1.0);
    return (1.0-t)*color(1.0, 1.0, 1.0) + t*color(0.5, 0.7, 1.0);
}
```

## 5. Adding a Sphere

```CPP
bool hit_sphere(const point3& center, double radius, const ray& r) {
    vec3 oc = r.origin() - center;
    auto a = dot(r.direction(), r.direction());
    auto b = 2.0 * dot(oc, r.direction());
    auto c = dot(oc, oc) - radius*radius;
    auto discriminant = b*b - 4*a*c;
    return (discriminant > 0);
}

color ray_color(const ray& r) {
    if (hit_sphere(point3(0,0,-1), 0.5, r))
        return color(1, 0, 0);
    vec3 unit_direction = unit_vector(r.direction());
    auto t = 0.5*(unit_direction.y() + 1.0);
    return (1.0-t)*color(1.0, 1.0, 1.0) + t*color(0.5, 0.7, 1.0);
}
```

Here we simply compute the discriminant of the quadratic equation to decide if the ray intersect with the sphere. However, the ray cannot see things "behind" (where `t < 0`) it. This is not a feature! We’ll fix those issues next.

## 6. Surface Normals and Multiple Objects

**Surface normals** used for _shading_. We don’t have any lights or anything yet, so just visualize the normals with a color map now.

A common trick used for **visualizing normals** (because it’s easy and somewhat intuitive to assume $\boldsymbol{n}$ s a **unit length** vector — so each component is between $−1$ and $1$) is to **map each component to the interval from $0$ to $1$, and then map `x/y/z` to `r/g/b`**

```CPP
auto t = hit_sphere(point3(0, 0, -1), 0.5, r);
// sphere
if (t > 0.0) {
    vec3 N = unit_vector(r.at(t) - vec3(0, 0, -1));
    return 0.5*color(N.x()+1, N.y()+1, N.z()+1);
}
// background
vec3 unit_direction = unit_vector(r.direction());
t = 0.5*(unit_direction.y() + 1.0);
return (1.0-t)*color(1.0, 1.0, 1.0) + t*color(0.5, 0.7, 1.0);
```

To get the **normal**, we need the **hit point**, **not** _just whether we hit or not_ (**discriminant** in the previous section). Here we still won't worry about negative values of `t` yet. We'll just assume the closest hit point (smallest `t`).

```CPP
if (discriminant < 0) {
    return -1.0;
} else {
    return (-b - sqrt(discriminant) ) / (2.0*a);
}
```

When `b = 2h`, the equation can be **simplified**:

$$
\frac{-b \pm \sqrt{b^2 - 4ac}}{2a} = \frac{-h \pm \sqrt{h^2 - ac}}{a}
$$

Using these observations, we can modify our code to save some computation:

```CPP
vec3 oc = r.origin() - center;
auto a = r.direction().length_squared();
auto half_b = dot(oc, r.direction());
auto c = oc.length_squared() - radius*radius;
auto discriminant = half_b*half_b - a*c;

if (discriminant < 0) {
    return -1.0;
} else {
    return (-half_b - sqrt(discriminant) ) / a;
}
```

What if we want more spheres? Compare to have an array of spheres, a very clean solution is the make an **“abstract class” for anything a ray might hit**, and make _both a sphere and a list of spheres just something you can hit_.

How to name the class? “object” (OOP)? “Surface” (volumes)? “hittable”?

```CPP
struct hit_record {
    point3 p;
    vec3 normal;
    double t;
};

class hittable {
    public:
        virtual bool hit(const ray& r, double t_min, double t_max, hit_record& rec) const = 0;
};
```

We can set things up to make **normals** satisfy one of following condition:

- always point _**"outward"** from the surface_
- always point _**against** the incident ray_
    
This decision is determined by **when** you want to **determine the side of the surface**:

- **geometry intersection**
- **coloring**

In this book we have more material types than we have geometry types, so we'll go for less work and put the determination at geometry time.

```CPP
bool front_face;

inline void set_face_normal(const ray& r, const vec3& outward_normal) {
    front_face = dot(r.direction(), outward_normal) < 0;
    normal = front_face ? outward_normal : -outward_normal;
}
```

We now add a class that stores a list of `hittable`s:

```CPP
class hittable_list : public hittable {
    public:
        hittable_list() {}
        hittable_list(shared_ptr<hittable> object) { add(object); }

        void clear() { objects.clear(); }
        void add(shared_ptr<hittable> object) { objects.push_back(object); }

        virtual bool hit(
            const ray& r, double t_min, double t_max, hit_record& rec) const override;

    public:
        std::vector<shared_ptr<hittable>> objects;
};
```

```ad-question
title: **\[Doubt\]** 🤔 Why the `hittable_list` class _inherits from_ `hittable` class?

We see that `hittable_list` class uses `hittable` class in its data and inherits from it in the meanwhile. I think the reason is `hittable_list` class is also a "hittable object" and it needs to implement the **`hit`** function (important) while the list can be composed of many different "single" hittable object (even another `hittable_list` object, kind of like recursion?)
The hittable object list can also be seen as a "single" object, it's up to how you define what is a "hittable object".
```

The following implement of **`hittablelist::hit`** shows _why we need `t_min` and `t_max`_ parameters in the hit function. We want to compute **the nearest intersection** in the hittable list, travese the objects in the list and use closet `t` has been computed so far as the `t_max` parameter to update the closet `t`.

```CPP
bool hittable_list::hit(const ray& r, double t_min, double t_max, hit_record& rec) const {
    hit_record temp_rec;
    bool hit_anything = false;
    auto closest_so_far = t_max;

    for (const auto& object : objects) {
        if (object->hit(r, t_min, closest_so_far, temp_rec)) {
            hit_anything = true;
            closest_so_far = temp_rec.t;
            rec = temp_rec;
        }
    }

    return hit_anything;
}
```

```ad-info
 **`shared_ptr<type>`** is a **pointer** to _some allocated type_, _with reference-counting_ semantics.

- Typically, a shared pointer is first _initialized with a newly-allocated object_
	- Every time you **assign** its value to another shared pointer (usually with a simple assignment), the _reference count is incremented_
	- As shared pointers **go out of scope** (like at the end of a block or function), the _reference count is decremented_
	- Once the **count goes to zero**, the object is _deleted_

`std::shared_ptr` is included with the `<memory>` header.
```

Now we can modify our _main.cpp_, remove the `ray_sphere` function, add two spheres to our world objects list, modify the `ray_color` function to compute the intersections and colors using `hit` function provided by `hittable` class.

## 7. Antialiasing

We can do **antialiasing** by _generating pixels with **multiple samples**_ in ray tracing.

One thing we need is a **random number generator** that returns **real** random numbers. We need a function that returns a canonical random number which by convention returns a random real $r$ in the range $[0, 1]$. _The “less than” before the 1 is important as we will sometimes take advantage of that_.

```CPP
#include <random>

inline double random_double() {
    static std::uniform_real_distribution<double> distribution(0.0, 1.0);
    static std::mt19937 generator;
    return distribution(generator);
}
```

```ad-info
**`std::uniform_real_distribution`** produces random floating-point values `x`, _uniformly_ distributed on the interval `[a, b)`.
```

```ad-info
`std::mt19937` (since C++11) class is a _very efficient pseudo-random number generator_. It produces 32-bit pseudo-random numbers using the well-known and popular algorithm named Mersenne twister algorithm. `std::mt19937` class is basically a type of `std::mersenne_twister_engine` class.
```

```ad-info
 The `<random>` library allows to produce random numbers using combinations of generators and distributions:

- **Generators**: Objects that generate uniformly distributed numbers
- **Distributions**: Objects that transform sequences of numbers generated by a generator into sequences of numbers that follow a specific random variable distribution, such as uniform, Normal or Binomial
```

## 8. Diffuse Materials

Now we can make some **realistic** looking materials. We’ll start with **diffuse (matte) materials**. That is, a **dull finish**, as on glass, metal, or paper.

One question is whether we choose one of:

- **mix** and **match** _geometry_ and _materials_ (so we can assign a material to multiple spheres, or vice versa)
- _geometry_ and _material_ are **tightly bound** (that could be useful for procedural objects where the geometry and material are linked)

We’ll go with the first option: **separate geometry and materials**.

```ad-note
title: **\[Obversvation\]** 🧐
The speed of rendering drops rapidly after adding recursion in `ray_color()`!
``` 

```ad-info
**Gamma correction** is a _nonlinear operation_ used to encode and decode luminance or tristimulus values in video or still image systems. Because _Our eyes do not perceive light the way cameras do_. Gamma encoding of images is used to _optimize the usage of bits when encoding an image, or bandwidth used to transport an image_. In the simplest cases, gamma correction is defined by the following power-law expression: $V_{\text{out}}=AV_{\text{in}}^{\gamma }$
```

An hack, change `t_min` to `0.001` for ignoring hits very near zero:

```CPP
if (world.hit(r, 0.001, infinity, rec)) {
```

**\[1\]** 📙 The rejection method presented here produces **random vector in unit sphere** _offset along the surface normal_:

```CPP
vec3 random_in_unit_sphere() {
    while (true) {
        auto p = vec3::random(-1,1);
        if (p.length_squared() >= 1) continue;
        return p;
    }
}
```

```CPP
point3 target = rec.p + rec.normal + random_in_unit_sphere();
return 0.5 * ray_color(ray(rec.p, target - rec.p), world);
```

This _corresponds to _picking directions on the_ **hemisphere** with **high probability close to the normal**, and a **lower probability of scattering rays at grazing angles**. This distribution scales by the $cos^3(\phi)$ where $\phi$ is the angle from the normal. This is useful since _light arriving at shallow angles **spreads over a larger area**, and thus has a **lower contribution** to the final color_.

![](https://raw.githubusercontent.com/binwatch/images/main/rayTracingIn1Weekend_0.png)

```ad-question
title: **\[Doubt\]** 🤔 Why $cos^3(\phi)$? How to prove it?
```

**\[2\]** 📙 However, we are interested in a Lambertian distribution, which has a distribution of $cos(\phi)$. **True Lambertian** has the probability higher for ray scattering close to the normal, but the _distribution is more **uniform**__. This is achieved by picking random points **on the surface of the unit sphere**, offset along the surface normal. Picking random points on the unit sphere can be achieved by picking random points _in_ the unit sphere, and then **normalizing** those:

```CPP
vec3 random_unit_vector() {
    return unit_vector(random_in_unit_sphere());
}
```

```CPP
point3 target = rec.p + rec.normal + random_unit_vector();
```

```ad-question
title: **\[Doubt\]** 🤔 Why $cos(\phi)$? How to prove it?
```

```ad-question
title: **\[Doubt\]** 🤔 Why this method (or hack?) can generate random vector (finally the reflected ray) with correct distribution?

```
 
**\[Compare 1 \& 2\]** 📘 There are two important visual differences between the two approaches:

1. The **shadows are less pronounced** in true Lambertian
2. Both spheres are **lighter** in appearance in true Lambertian

Both of these changes are _due to the **more uniform scattering** of the light rays_, fewer rays are scattering toward the normal. This means that for diffuse objects, they will appear _lighter_ because _more light bounces toward the camera_. For the shadows, less light bounces straight-up (_more likey to hit the background instead of the spheres_), so the parts of the larger sphere directly underneath the smaller sphere are brighter.

**\[3\]** 📙 Another more intuitive approach is to have **a uniform scatter direction for all angles** away from the hit point, with _no dependence on the angle from the normal_.

```ad-question
title: **\[Doubt\]** 🤔 I noticed that the three methods take successively less time, probably because the number of light bounces is reduced?
```

## 9. Metal

If we want different objects to have different materials, here are 2 design decisions:

- A **universal material** with lots of parameters and different material types just zero out some of those parameters
- An **abstract material class** that encapsulates behavior

We choose the latter approach. For our program the material needs to do two things:

1. Produce a **scattered** ray (or say it **absorbed** the incident ray)
2. If scattered, say how much the ray should be **attenuated**

```ad-question
title: **\[Doubt\]** 🤔 Why placing the `shared_ptr<material>` data in `sphere` class instead of `hittable` class?

The `hittable_list` class also (public) inherits from `hittable` class, but it composed of multiple objects, each one can has a different material type, we cannot use `shared_ptr<material>` data in it.
```

```ad-info
**Albedo** (from Latin _albedo_ '**whiteness**') is the measure of the **diffuse reflection of solar radiation out** _of_ the **total solar radiation**. It is measured on a scale from $0$ (corresponding to a black body that **absorbs** _all incident radiation_) to $1$ (corresponding to a body that **reflects** _all incident radiation_).

It determines the color of the object, so we use `color` to express it in our code.
``` 

Add a fuzziness parameter to **randomize the reflected direction** by using a small sphere and choosing a new endpoint for the ray, the bigger the sphere, the fuzzier the reflections will be (_is it glossy metal meterial?_).

```CPP
metal(const color& a, double f) : albedo(a), fuzz(f < 1 ? f : 1) {}
```

````ad-note
We do not need to check if `f < 0` here, because the code in `metal::scatter`:

```CPP
scattered = ray(rec.p, reflected + fuzz*random_in_unit_sphere());
```

add `fuzz * random vector in unit sphere` to the reflected, it's spherically symmetric
````

```ad-question
title: **\[Doubt\]** 🤔 But, why don't we check `fabs(f) < 1.0` here instead?
```

## 10. Dielectrics

Clear materials such as water, glass, and diamonds are **dielectrics**. When a light ray hits them, it splits into a **reflected** ray and a **refracted** (transmitted) ray.

We’ll handle that by _randomly choosing between reflection or refraction_, and _only generating one scattered ray per interaction_.

In a shpere with dielectrics material the world should be **flipped upside down** because of refraction.

One troublesome practical issue is that when the ray is in the material with the **higher refractive index, there is no _real_ solution to Snell’s law**.

For example, when index of refraction is greater then $1.0$, here is $1.5$, if:

$$
1.5 \sin{\theta} > 1.0
$$

Then the equation

$$
\sin{\theta^{\prime}} = \frac{\eta}{\eta^{\prime}} \sin{\theta}
$$

doesn't have **real** solution.

Real glass has **reflectivity that varies with angle**: look at a window at a **steep angle** and it becomes a **mirror** (_more reflection than refraction_). There is a big ugly equation for that, but almost everybody uses a cheap and surprisingly accurate polynomial approximation by Christophe Schlick:

```CPP
class dielectric : public material {
    public:
        dielectric(double index_of_refraction) : ir(index_of_refraction) {}

        virtual bool scatter(
            const ray& r_in, const hit_record& rec, color& attenuation, ray& scattered
        ) const override {
            attenuation = color(1.0, 1.0, 1.0);
            double refraction_ratio = rec.front_face ? (1.0/ir) : ir;

            vec3 unit_direction = unit_vector(r_in.direction());
            double cos_theta = fmin(dot(-unit_direction, rec.normal), 1.0);
            double sin_theta = sqrt(1.0 - cos_theta*cos_theta);

            bool cannot_refract = refraction_ratio * sin_theta > 1.0;
            vec3 direction;
            if (cannot_refract || reflectance(cos_theta, refraction_ratio) > random_double())
                direction = reflect(unit_direction, rec.normal);
            else
                direction = refract(unit_direction, rec.normal, refraction_ratio);

            scattered = ray(rec.p, direction);
            return true;
        }

    public:
        double ir; // Index of Refraction

    private:
        static double reflectance(double cosine, double ref_idx) {
            // Use Schlick's approximation for reflectance.
            auto r0 = (1-ref_idx) / (1+ref_idx);
            r0 = r0*r0;
            return r0 + (1-r0)*pow((1 - cosine),5);
        }
};
```

```ad-question
title: **\[Doubt\]** 🤔 Looks cool! But why it works?
```

An interesting and easy trick with dielectric spheres is to note that if you _use a negative radius_, the _geometry is unaffected_, but the _surface normal points inward_. This can be used as a bubble to make a **hollow** glass sphere

````ad-question
title: **\[Doubt\]** 🤔 Why a hollow glass sphere can be made by this method?

Negative radius just causes inward surface normal:

```CPP
vec3 outward_normal = (rec.p - center) / radius;
```

The index of refraction is depends on the substances on both sides, so the inward surface normal inward then "swap" the substances:

```CPP
double refraction_ratio = rec.front_face ? (1.0/ir) : ir;
```
````
 
## 11. Positionable Camera

To get an **arbitrary viewpoint**, we need a camera that can **move, rotate and adjust field of view**. First consider moving, here is our naming convention:

- `lookfrom`: the position where we place the camera
- `lookat`: the point we look at

```ad-question
title: **\[Doubt\]** 🤔 What is the "camera" defined? the lens (in next section) or the pinhole or the sensor (maybe it is the "opposite" project plane, which is `lookat` on)?
```

We also need a way to specify the roll, or sideways tilt, of the camera: the rotation around the lookat-lookfrom axis. Another way to think about it is that even if you keep `lookfrom` and `lookat` constant, you can still rotate your head around your nose. What we need is a way to specify an “up” vector for the camera. This up vector should lie in the plane orthogonal to the view direction. A common convention of naming is “view up” (`vup`) vector.

```ad-note
Our camera has **3 degrees of freedom**, as it is a rigid body: `lookfrom`, `lookat` (or `lookfrom-lookat` vector) and `vup`
```

```ad-note
Notice that `vup` is **not** equal to `v`! Actually `vup` is used to decide `u` then `v` using cross product
```

## 12. Defocus Blur

**Defocus blur** occurs in real cameras, photographers will call it "**depth of field**". The reason is real cameras need a _big hole_ rather than just a pinhole to _gather light_, which would _defocus everything_. Cameras stick a [**lens**](https://en.wikipedia.org/wiki/Lens) in the hole to slove this problem, all light rays coming _from_ a specific point at the **focus distance** hit the lens will be bent back _to_ a single point on the image sensor (the distance between image sensor and the lens is the same as focus distance?).

```ad-info
**focus distance** is the _distance_ between the _projection point_ and the _plane_ where everything is in **perfect focus**.

In a physical camera, the focus distance is controlled by the distance between the lens and the film/sensor. That is why you see the lens move relative to the camera when you change what is in focus (that may happen in your phone camera too, but the sensor moves).
```

```ad-info
**aperture** is a _hole_ to _control how big the lens is effectively_.

Bigger aperture, more light and more defocus blur (wider range to accept light) in the real camera.
```

We could simulate the order:

1. sensor
2. lens
3. aperture

We don't need to simulate the inside of the camera for the purposes of rendering an image outside the camera.

We can start rays _from_ the lens and send them toward the focus plane (`focus_dist` away from the lens) because of light path's invertable property.

In order to accomplish defocus blur, generate random scene rays originating _from_ inside **a disk centered at the** **`lookfrom`** **point**. The _larger the radius_, the _greater the defocus blur_.

```CPP
horizontal = focus_dist * viewport_width * u;
vertical = focus_dist * viewport_height * v;
lower_left_corner = origin - horizontal/2 - vertical/2 - focus_dist*w;

lens_radius = aperture / 2;
```

```ad-question
title: **\[Doubt\]** 🤔 Why considering lens' focus distance streches the size of viewport and focal length by `focus_dist` (does not change the fov) here?
```

```CPP
ray get_ray(double s, double t) const {
    vec3 rd = lens_radius * random_in_unit_disk();
    vec3 offset = u * rd.x() + v * rd.y();

    return ray(
        origin + offset,
        lower_left_corner + s*horizontal + t*vertical - origin - offset
    );
}
```

Here we use sampling method to simulate the circle of confusion (defocus blur effect), multiple rays distributed across the aperture range enter one point.

```ad-question
title: **\[Doubt\]** 🤔 Why the code works correct intuitively?
``` 

## 13. A Final Render

> **[Imporve?]** 🚀 The rendering process is really slow! This simple ray tracer has to traverse every object in the sence for each ray! However, the workload can be deduced using some data struct to manage the objects such as BVH. And we can also use SIMP, SIMD on multi-core CPU or the powerful GPU to deal with large repetitivework.
> Use PBR to make our results more realistic.

## 14. Next Steps

### Lights

> You can do this explicitly, by sending shadow rays to lights, or it can be done implicitly by making some objects emit light, biasing scattered rays toward them, and then downweighting those rays to cancel out the bias. Both work. I am in the minority in favoring the latter approach.

### Triangles

> Most cool models are in triangle form. The model I/O is the worst and almost everybody tries to get somebody else’s code to do this.

### Surface Textures

> his lets you paste images on like wall paper. Pretty easy and a good thing to do.

### Solid textures

> Ken Perlin has his code online. Andrew Kensler has some very cool info at his blog.

### Volumes and Media

> Cool stuff and will challenge your software architecture. I favor making volumes have the hittable interface and probabilistically have intersections based on density. Your rendering code doesn’t even have to know it has volumes with that method.

### Parallelism

> Run $N$ copies of your code on $N$  cores with different random seeds. Average the $N$ runs. This averaging can also be done hierarchically where $N/2$ pairs can be averaged to get $N/4$ images, and pairs of those can be averaged. That method of parallelism should extend well into the thousands of cores with very little coding.




