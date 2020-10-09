/**
 * 程序名：前端路由配置
 * 功能：配置前端路由
 */
import Vue from "vue";
import Router from "vue-router";
import Login from "@/components/Login";
import Base from "@/components/Base";
import Home from "@/components/Home";
import Display from "@/components/Display";
import ThankYou from "@/components/ThankYou";
import Menu from "@/components/Management/Menu";
import User from "@/components/Management/User";
import ChangePassword from "@/components/ChangePassword";
import AuditPaper from "@/components/Management/AuditPaper";
import Question from "@/components/Management/Question";
import Classification from "@/components/Management/Classification";
import Modular from "@/components/Management/Modular";
import ProblemData from "@/components/Management/ProblemData";
import TempDisplay from "@/components/tempDisplay";
Vue.use(Router);

export default new Router({
  //打包时候注解
  mode: "history",
  routes: [
    {
      path: "/",
      name: "Base",
      component: Base,
      children: [
        {
          path: "/",
          name: "Login",
          component: Login
        },
        {
          path: "/login",
          name: "Login",
          component: Login
        },
        {
          path: "/home",
          name: "Home",
          component: Home
        },
        {
          path: "/management",
          name: "Menu",
          component: Menu,
          children: [
            {
              path: "/management/user",
              name: "User",
              component: User
            },
            {
              path: "/management/auditPaper",
              name: "AuditPaper",
              component: AuditPaper
            },
            {
              path: "/management/question",
              name: "Question",
              component: Question
            },
            {
              path: "/management/classification",
              name: "Classification",
              component: Classification
            },
            {
              path: "/management/modular",
              name: "Modular",
              component: Modular
            },
            {
              path: "/management/problemData/:sn",
              name: "ProblemData",
              component: ProblemData
            }
          ]
        },
        {
          path: "/changePassword",
          name: "ChangePassword",
          component: ChangePassword
        }
      ]
    },
    {
      path: "/display/:sn",
      name: "Display",
      component: Display
    },
    {
      path: "/tempDisplay/:sn",
      name: "TempDisplay",
      component: TempDisplay
    },
    {
      path: "/thankyou",
      name: "ThankYou",
      component: ThankYou
    }
  ]
});
