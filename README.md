# blogapp

main.dart
import 'package:blogapp/screen/splash_screen.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:flutter/material.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Flutter Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
            seedColor: const Color.fromARGB(255, 255, 89, 219)),
        useMaterial3: true,
      ),
      home: SplashScreen(),
    );
  }
}

splash_screen.dart
import 'dart:async';

import 'package:blogapp/screen/home_page.dart';
import 'package:blogapp/screen/option_Screen.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'package:flutter/material.dart';

class SplashScreen extends StatefulWidget {
  const SplashScreen({super.key});

  @override
  State<SplashScreen> createState() => _SplashScreenState();
}

class _SplashScreenState extends State<SplashScreen> {
  final auth = FirebaseAuth.instance;

  @override
  void initState() {
    final user = auth.currentUser;
    if (user != null) {
      Timer(Duration(seconds: 3), () {
        Navigator.push(
            context, MaterialPageRoute(builder: (context) => BlogPage()));
      });
    } else {
      Timer(Duration(seconds: 3), () {
        Navigator.push(
            context, MaterialPageRoute(builder: (context) => OptionScreen()));
      });
    }
    // TODO: implement initState
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        crossAxisAlignment: CrossAxisAlignment.center,
        children: [
          Image.network(
              'https://tse2.mm.bing.net/th?id=OIP.GMD2i3A6NvDebjZB3wPcoQHaHa&pid=Api&P=0&h=180'),
          Align(alignment: Alignment.center, child: Text("Blog App"))
        ],
      ),
    );
  }
}
HomePage or Blog Page
import 'package:blogapp/components/utils.dart';
import 'package:blogapp/screen/login_screen.dart';
import 'package:firebase_database/firebase_database.dart';
import 'package:firebase_database/ui/firebase_animated_list.dart';
import 'package:flutter/material.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'package:flutter/services.dart';

import 'uploadimage.dart';

class BlogPage extends StatefulWidget {
  const BlogPage({super.key});

  @override
  State<BlogPage> createState() => _BlogPageState();
}

class _BlogPageState extends State<BlogPage> {
  final ref = FirebaseDatabase.instance.ref().child("Post");
  final auth = FirebaseAuth.instance;
  final searchController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return WillPopScope(
      onWillPop: () async {
        SystemNavigator.pop();
        return true;
      },
      child: Scaffold(
        appBar: AppBar(
          centerTitle: true,
          actions: [
            Padding(
              padding: const EdgeInsets.only(right: 10),
              child: IconButton(
                  onPressed: () {
                    Navigator.push(context,
                        MaterialPageRoute(builder: (context) => UploadImage()));
                  },
                  icon: Icon(
                    Icons.add,
                    size: 30,
                    color: Colors.white,
                  )),
            ),
            IconButton(
                onPressed: () {
                  auth.signOut().then((value) {
                    Util().toastMessage('You are Logout');
                    Navigator.push(context,
                        MaterialPageRoute(builder: (context) => LoginScreen()));
                  }).onError((error, stackTrace) {
                    Util().toastMessage(error.toString());
                  });
                },
                icon: Icon(Icons.logout))
          ],
          automaticallyImplyLeading: false,
          backgroundColor: Theme.of(context).colorScheme.inversePrimary,
          title: Text("Blog Pages"),
        ),
        body: Padding(
          padding: const EdgeInsets.all(8.0),
          child: Column(
            children: [
              TextField(
                controller: searchController,
                onChanged: (value) {
                  setState(() {});
                },
                decoration: InputDecoration(
                    border: OutlineInputBorder(),
                    prefixIcon: Icon(Icons.search),
                    hintText: 'Search'),
              ),
              Expanded(
                  child: FirebaseAnimatedList(
                      physics: BouncingScrollPhysics(),
                      query: ref.child('Post List'),
                      itemBuilder: (BuildContext context, DataSnapshot snapshot,
                          Animation<double> animation, index) {
                        final title = snapshot.child('ptitle').value.toString();
                        if (searchController.text.isEmpty) {
                          return Padding(
                            padding: const EdgeInsets.all(8.0),
                            child: Container(
                              decoration: BoxDecoration(
                                  color: Colors.grey.shade300,
                                  borderRadius: BorderRadius.circular(10)),
                              child: Column(
                                children: [
                                  ClipRRect(
                                    borderRadius: BorderRadius.circular(10),
                                    child: FadeInImage.assetNetwork(
                                        fit: BoxFit.cover,
                                        height:
                                            MediaQuery.of(context).size.height *
                                                .25,
                                        width:
                                            MediaQuery.of(context).size.width *
                                                1,
                                        placeholder: 'lib/image/M.png',
                                        image: snapshot
                                            .child('pUrl')
                                            .value
                                            .toString()),
                                  ),
                                  Text(
                                    snapshot.child('ptitle').value.toString(),
                                    style: TextStyle(fontSize: 18),
                                  ),
                                  Text(
                                    snapshot
                                        .child('pdiscription')
                                        .value
                                        .toString(),
                                    style: TextStyle(fontSize: 17),
                                  )
                                ],
                              ),
                            ),
                          );
                        } else if (title.toUpperCase().toString().contains(
                            searchController.text.toString().toLowerCase())) {
                          return Padding(
                            padding: const EdgeInsets.all(8.0),
                            child: Container(
                              decoration: BoxDecoration(
                                  color: Colors.grey.shade300,
                                  borderRadius: BorderRadius.circular(10)),
                              child: Column(
                                children: [
                                  ClipRRect(
                                    borderRadius: BorderRadius.circular(10),
                                    child: FadeInImage.assetNetwork(
                                        fit: BoxFit.cover,
                                        height:
                                            MediaQuery.of(context).size.height *
                                                .25,
                                        width:
                                            MediaQuery.of(context).size.width *
                                                1,
                                        placeholder: 'lib/image/M.png',
                                        image: snapshot
                                            .child('pUrl')
                                            .value
                                            .toString()),
                                  ),
                                  Text(
                                    snapshot.child('ptitle').value.toString(),
                                    style: TextStyle(fontSize: 18),
                                  ),
                                  Text(
                                    snapshot
                                        .child('pdiscription')
                                        .value
                                        .toString(),
                                    style: TextStyle(fontSize: 17),
                                  )
                                ],
                              ),
                            ),
                          );
                        } else {
                          return Container();
                        }
                      }))
            ],
          ),
        ),
      ),
    );
  }
}

Upload Image
import 'dart:io';

import 'package:blogapp/components/round_button.dart';
import 'package:blogapp/components/utils.dart';
import 'package:firebase_storage/firebase_storage.dart';
import 'package:flutter/material.dart';
import 'package:image_picker/image_picker.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'package:firebase_storage/firebase_storage.dart' as firebase_storage;
import 'package:firebase_database/firebase_database.dart';

class UploadImage extends StatefulWidget {
  const UploadImage({super.key});

  @override
  State<UploadImage> createState() => _UploadImageState();
}

class _UploadImageState extends State<UploadImage> {
  final postref = FirebaseDatabase.instance.ref().child('Post');
  firebase_storage.FirebaseStorage storage =
      firebase_storage.FirebaseStorage.instance;
  final auth = FirebaseAuth.instance;
  final _formKey = GlobalKey<FormState>();
  final titleController = TextEditingController();
  final discriptionController = TextEditingController();
  File? image;
  final picker = ImagePicker();
  Future getGalleryImage() async {
    final pickedFile = await picker.pickImage(source: ImageSource.gallery);
    setState(() {
      if (pickedFile != null) {
        image = File(pickedFile.path);
      } else {
        print('no pick any file');
      }
    });
  }

  Future getCameraImage() async {
    final pickedFile = await picker.pickImage(source: ImageSource.camera);
    setState(() {
      if (pickedFile != null) {
        image = File(pickedFile.path);
      } else {
        print('no pick any file');
      }
    });
  }

  void dialog() {
    showDialog(
        context: context,
        builder: (BuildContext context) {
          return AlertDialog(
            shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(10.0)),
            content: Container(
              height: 120,
              child: Column(
                children: [
                  InkWell(
                    onTap: () {
                      getCameraImage();
                      Navigator.pop(context);
                    },
                    child: ListTile(
                      title: Text("Camera"),
                      leading: Icon(Icons.camera),
                    ),
                  ),
                  InkWell(
                    onTap: () {
                      getGalleryImage();
                      Navigator.pop(context);
                    },
                    child: ListTile(
                      title: Text("Gallery"),
                      leading: Icon(Icons.browse_gallery),
                    ),
                  )
                ],
              ),
            ),
          );
        });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        centerTitle: true,
        title: Text("Upload Image"),
      ),
      body: Padding(
        padding: const EdgeInsets.all(8.0),
        child: Form(
          key: _formKey,
          child: SingleChildScrollView(
            child: Column(
              children: [
                InkWell(
                  onTap: () {
                    dialog();
                  },
                  child: Center(
                    child: Padding(
                      padding: const EdgeInsets.all(8.0),
                      child: Container(
                        height: 130,
                        width: 400,
                        child: image != null
                            ? ClipRect(
                                child: Image.file(
                                  image!.absolute,
                                  height: 100,
                                  width: 100,
                                  fit: BoxFit.cover,
                                ),
                              )
                            : Container(
                                height: 100,
                                width: 100,
                                decoration: BoxDecoration(
                                    borderRadius: BorderRadius.circular(10),
                                    color: Colors.grey.shade200),
                                child: Icon(
                                  Icons.camera_alt,
                                  color: Theme.of(context)
                                      .colorScheme
                                      .inversePrimary,
                                ),
                              ),
                      ),
                    ),
                  ),
                ),
                SizedBox(
                  height: 20,
                ),
                TextFormField(
                  controller: titleController,
                  decoration: InputDecoration(
                    border: OutlineInputBorder(),
                    label: Text("Title"),
                    hintText: 'Title',
                  ),
                  validator: (value) {
                    if (value == null || value.isEmpty) {
                      return 'Enter title';
                    }
                    return null;
                  },
                ),
                SizedBox(
                  height: 20,
                ),
                TextFormField(
                  minLines: 1,
                  maxLines: 5,
                  controller: discriptionController,
                  decoration: InputDecoration(
                      label: Text("Enter Discription"),
                      hintText: 'Discription',
                      border: OutlineInputBorder()),
                ),
                SizedBox(
                  height: 10,
                ),
                RoundButton(
                    title: "Upload Image",
                    onPressed: () async {
                      try {
                        int date = DateTime.now().microsecondsSinceEpoch;
                        firebase_storage.Reference ref = firebase_storage
                            .FirebaseStorage.instance
                            .ref('/blogapp$date');
                        UploadTask uploadTask = ref.putFile(image!.absolute);
                        await Future.value(uploadTask);
                        var newUrl = await ref.getDownloadURL();
                        final user = auth.currentUser;
                        postref.child('Post List').child(date.toString()).set({
                          'pid': date.toString(),
                          'pUrl': newUrl.toString(),
                          'ptitle': titleController.text.toString(),
                          'pdiscription': discriptionController.text.toString(),
                          'pEmail': user!.email.toString()
                        }).then((value) {
                          Util().toastMessage('Upload Successful');
                        }).onError((error, stackTrace) {
                          Util().toastMessage(error.toString());
                        });
                      } catch (e) {
                        Util().toastMessage(e.toString());
                      }
                    }),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
login_screen.dart
import 'package:blogapp/components/round_button.dart';
import 'package:blogapp/components/utils.dart';
import 'package:blogapp/screen/forgot_password.dart';
import 'package:blogapp/screen/home_page.dart';
import 'package:blogapp/screen/register_screen.dart';
import 'package:flutter/material.dart';
import 'package:firebase_auth/firebase_auth.dart';

class LoginScreen extends StatefulWidget {
  const LoginScreen({super.key});

  @override
  State<LoginScreen> createState() => _ReiesterPageState();
}

class _ReiesterPageState extends State<LoginScreen> {
  final emailController = TextEditingController();
  final passwordController = TextEditingController();
  final formKey = GlobalKey<FormState>();
  final _auth = FirebaseAuth.instance;
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        centerTitle: true,
        automaticallyImplyLeading: false,
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: Text("Login"),
      ),
      body: SingleChildScrollView(
        child: Form(
          key: formKey,
          child: Padding(
            padding: const EdgeInsets.all(8.0),
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              crossAxisAlignment: CrossAxisAlignment.center,
              children: [
                SizedBox(
                  height: 100,
                ),
                TextFormField(
                  controller: emailController,
                  keyboardType: TextInputType.emailAddress,
                  decoration: InputDecoration(
                      border: OutlineInputBorder(),
                      prefixIcon: Icon(Icons.email),
                      label: Text('Email'),
                      hintText: 'Email'),
                  validator: (value) {
                    if (value == null || value.isEmpty) {
                      return 'Please Enter Your Email';
                    } else if (!value.contains('@')) {
                      return 'Please Enter Valid Email';
                    }
                    return null;
                  },
                ),
                SizedBox(
                  height: 20,
                ),
                TextFormField(
                  controller: passwordController,
                  obscureText: true,
                  obscuringCharacter: 'X',
                  decoration: InputDecoration(
                      border: OutlineInputBorder(),
                      prefixIcon: Icon(Icons.lock_clock_outlined),
                      label: Text('Password'),
                      hintText: 'Password'),
                  validator: (value) {
                    if (value == null || value.isEmpty) {
                      return 'Please Enter Your Password';
                    } else if (value.length < 6) {
                      return 'Please Enter 6 digit Password';
                    }
                    return null;
                  },
                ),
                SizedBox(
                  height: 10,
                ),
                Align(
                    alignment: Alignment.centerRight,
                    child: TextButton(
                        onPressed: () {
                          Navigator.push(
                              context,
                              MaterialPageRoute(
                                  builder: (context) => ForgotPassword()));
                        },
                        child: Text('Forgot Password'))),
                SizedBox(
                  height: 10,
                ),
                RoundButton(
                    title: 'Login',
                    onPressed: () async {
                      if (formKey.currentState!.validate()) {
                        setState(() {});
                        try {
                          final user = await _auth.signInWithEmailAndPassword(
                              email: emailController.text.toString(),
                              password: passwordController.text.toString());
                          // ignore: unnecessary_null_comparison
                          if (user != null) {
                            print('successful Login');
                            Util().toastMessage('Successfull Login');
                            Navigator.push(
                                context,
                                MaterialPageRoute(
                                    builder: (context) => BlogPage()));
                          }
                        } catch (e) {
                          print(e.toString());
                          Util().toastMessage(e.toString());
                        }
                      }
                    }),
                Row(
                  mainAxisAlignment: MainAxisAlignment.end,
                  children: [
                    Text('I don\'t have any account '),
                    TextButton(
                        onPressed: () {
                          Navigator.push(
                              context,
                              MaterialPageRoute(
                                  builder: (context) => RegisterPage()));
                        },
                        child: Text("Register "))
                  ],
                )
              ],
            ),
          ),
        ),
      ),
    );
  }
}

signup_screen.dart
// ignore_for_file: unnecessary_null_comparison

import 'package:blogapp/components/round_button.dart';
import 'package:blogapp/components/utils.dart';
import 'package:blogapp/screen/home_page.dart';
import 'package:blogapp/screen/login_screen.dart';
import 'package:flutter/material.dart';
import 'package:firebase_auth/firebase_auth.dart';

class RegisterPage extends StatefulWidget {
  const RegisterPage({super.key});

  @override
  State<RegisterPage> createState() => _ReiesterPageState();
}

class _ReiesterPageState extends State<RegisterPage> {
  final emailController = TextEditingController();
  final passwordController = TextEditingController();
  final formKey = GlobalKey<FormState>();
  final _auth = FirebaseAuth.instance;
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        centerTitle: true,
        automaticallyImplyLeading: false,
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: Text("Create Account"),
      ),
      body: SingleChildScrollView(
        child: Form(
          key: formKey,
          child: Padding(
            padding: const EdgeInsets.all(8.0),
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              crossAxisAlignment: CrossAxisAlignment.center,
              children: [
                SizedBox(
                  height: 100,
                ),
                TextFormField(
                  controller: emailController,
                  keyboardType: TextInputType.emailAddress,
                  decoration: InputDecoration(
                      border: OutlineInputBorder(),
                      prefixIcon: Icon(Icons.email),
                      label: Text('Email'),
                      hintText: 'Email'),
                  validator: (value) {
                    if (value == null || value.isEmpty) {
                      return 'Please Enter Your Email';
                    } else if (!value.contains('@')) {
                      return 'Please Enter Valid Email';
                    }
                    return null;
                  },
                ),
                SizedBox(
                  height: 20,
                ),
                TextFormField(
                  controller: passwordController,
                  obscureText: true,
                  obscuringCharacter: 'X',
                  decoration: InputDecoration(
                      border: OutlineInputBorder(),
                      prefixIcon: Icon(Icons.lock_clock_outlined),
                      label: Text('Password'),
                      hintText: 'Password'),
                  validator: (value) {
                    if (value == null || value.isEmpty) {
                      return 'Please Enter Your Password';
                    } else if (value.length < 6) {
                      return 'Please Enter 6 digit Password';
                    }
                    return null;
                  },
                ),
                SizedBox(
                  height: 40,
                ),
                RoundButton(
                    title: 'Register',
                    onPressed: () async {
                      if (formKey.currentState!.validate()) {
                        setState(() {});
                        try {
                          final user =
                              await _auth.createUserWithEmailAndPassword(
                                  email: emailController.text.toString(),
                                  password: passwordController.text.toString());
                          if (user != null) {
                            print('successful Register');
                            Util().toastMessage('SuccessFul Register');
                            Navigator.push(
                                context,
                                MaterialPageRoute(
                                    builder: (context) => BlogPage()));
                          }
                        } catch (e) {
                          print(e.toString());
                          Util().toastMessage(e.toString());
                        }
                      }
                    }),
                Row(
                  mainAxisAlignment: MainAxisAlignment.end,
                  children: [
                    Text('All ready have a account'),
                    TextButton(
                        onPressed: () {
                          Navigator.push(
                              context,
                              MaterialPageRoute(
                                  builder: (context) => LoginScreen()));
                        },
                        child: Text('Login '))
                  ],
                )
              ],
            ),
          ),
        ),
      ),
    );
  }
}

